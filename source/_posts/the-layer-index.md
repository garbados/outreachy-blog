---
title: The Layer Index and Asynchronous Task Queues in Django
date: 2017-06-21 15:50:43
tags:
- yocto
- python
- django
---

Greetings!

I've been working full-time on the [Yocto Project](https://yoctoproject.org/) for a couple weeks now, and I'd like to talk about what I've been up to so far.

First, what is Yocto? It's a collection of FOSS projects for building Linux distributions for embedded systems, like RasberryPi boards. The distributions you create using tools like bitbake can then be imaged onto hardware, allowing you to (for example) quickly prepare routers with the necessary drivers and networking software to create a [mesh network](https://en.wikipedia.org/wiki/Mesh_networking). Outside of my internship, I hope to develop a production pipeline for inexpensive mesh nodes, somewhat like [PirateBox](https://piratebox.cc/), and tools like bitbake will be a big part of that effort.

My mentor, Libertad Cruz, showed me a handful of bugs on a project called [layerindex-web](https://layers.openembedded.org) which indexes layers and recipes on top of [OE-Core](https://www.openembedded.org/wiki/OpenEmbedded-Core). For example, there's the [meta-rasberrypi](https://git.yoctoproject.org/cgit/cgit.cgi/meta-raspberrypi/about/) layer which provides support for developing distributions for RasberryPi boards. A recipe might be a distribution you can further customize, or instructions for including certain software in a distribution.

layerindex-web is a Django application that allows users to submit layers. The application uses git repo addresses to infer information about a layer, including its recipes and branches. An interesting feature of the site is that maintainers have editing rights over the things they maintain, but a maintainer doesn't need an account on the site to be listed as a maintainer. Thus, non-maintainers can submit layers to the index, and the maintainer can create an account later in order to make any necessary edits, should they be so inclined. All layers require admin sign-off before they are published to the index.

The first issue I chose to tackle requested the implementation of an asynchronous task runner, so that tasks like sending emails or fetching layer information don't block page rendering. In some cases, this means requests might time out for taking too long, or if they error out, the page would fail to load. I chose to use [RabbitMQ](https://www.rabbitmq.com/) and [Celery](http://docs.celeryproject.org/en/latest/) to do the job. RabbitMQ is a message broker, like a specialized kind of database, which runs outside of Django. Celery is a distributed task queue written in Python. It runs inside Django, and communicates with RabbitMQ to manage the queue and execute tasks asynchronously.

Enough background. Here's how I implemented an asynchronous task queue in Python 3.5 and Django 1.8:

## Installing RabbitMQ

On systems with `apt-get` like Ubuntu, do this:

```bash
sudo apt-get install rabbitmq-server
```

This installs the RabbitMQ server, and starts the server daemon with some default settings. See [Configuring RabbitMQ](https://www.rabbitmq.com/configure.html) for more information about configurating settings. Within development and QA, the default settings might be sufficient for you. They are for me.

If you use Docker or Vagrant or Chef or some other kind of automated provisioning tool, you'll need to add instructions to include RabbitMQ in your build. You can find more information, including recipes, in [RabbitMQ's installation guides](https://www.rabbitmq.com/download.html).

Then, you'll need to include connection information in your application's `settings.py`. Here's how I did it:

```python
# RabbitMQ settings
RABBIT_BROKER = 'amqp://'
RABBIT_BACKEND = 'rpc://'
```

Those reflect RabbitMQ's default settings. The broker is the location of the message broker (RabbitMQ in this case), the backend is the protocol used to send messages to the broker. Celery will know what to do with these settings.

## Using Celery

Use [pip](https://pypi.python.org/pypi/pip) to install Celery, and add it to your `requirements.txt` file:

```bash
pip install celery
pip freeze > requirements.txt
```

Now create a `tasks.py` file in your project. The name is arbitrary, I just find the name appropriate for, y'know, writing tasks. Paste something like this as a starting point:

```python
# tasks.py
from celery import Celery
from django.core.mail import EmailMessage
from settings import RABBIT_BROKER, RABBIT_BACKEND
# Celery namespaces task queues, so you can
# theoretically run several under different names.
# To start, we'll only use one, named after your app.
APP_NAME = "the-name-of-your-application"
# Set up the task queue
tasks = Celery(APP_NAME, broker=RABBIT_BROKER, backend=RABBIT_BACKEND)
# Register a 'hello_world' task
# which can then be called via `tasks.hello_world()`
@tasks.task
def hello_world(txt="Hello, world!"):
	print(txt)
# For a more practical example, register a 'send_email' task
# which asynchronously sends an email:
@tasks.task
def send_email(subject, text_content, from_email, to_emails=[]):
	msg = EmailMessage(subject, text_content, from_email, to_emails)
	return msg.send()
```

From another file, such as views.py, you can import the tasks object and use it in your views. For example:

```python
# views.py
from django.contrib.auth.models import User, Permission
from django.core.urlresolvers import reverse
from django.db.models import Q
from django.http import HttpResponseRedirect
from django.template.loader import get_template
from forms import EditLayerForm, LayerMaintainerFormSet
from models import LayerItem, LayerBranch, Branch
from tasks import tasks
import settings
# Consider a view that emails admins
# whenever a new layer is submitted.
# This is a simplified version of the view
# used in layerindex-web.
def submit_layer_view(request, branch_name="master"):
  # Instantiate the objects that will be associated with the new layer.
	branchobj = Branch.objects.filter(name=branch)[:1].get()
	layeritem = LayerItem()
	layerbranch = LayerBranch(layer=layeritem, branch=branchobj)
  # Ensure submitted layer info is valid by passing it through forms.
	form = EditLayerForm(request.user, layerbranch, request.POST, instance=layeritem)
	maintainerformset = LayerMaintainerFormSet(request.POST, instance=layerbranch)
	if form.is_valid() and maintainerformset.is_valid():
		form.save()
		layerbranch.save()
		maintainerformset.save()
		# Send email to those who can publish layers
		plaintext = get_template('layerindex/submitemail.txt')
		perm = Permission.objects.get(codename='publish_layer')
		users = User.objects.filter(Q(groups__permissions=perm) | Q(user_permissions=perm) | Q(is_superuser=True) ).distinct()
		subject = '%s - %s' % (settings.SUBMIT_EMAIL_SUBJECT, layeritem.name)
		from_email = settings.SUBMIT_EMAIL_FROM
		for user in users:
			d = Context({
				'layer_name': layeritem.name,
				'layer_url': request.build_absolute_uri(reverse('layer_review', args=(layeritem.name,))),
				'user_name': user.first_name if user.first_name else user.username
			})
			text_content = plaintext.render(d)
			tasks.send_email(subject, text_content, from_email, [user.email])
		return HttpResponseRedirect(reverse('submit_layer_thanks'))
	else:
		return HttpResponse("The forms you submitted were invalid, my friend.", content_type="text/plain", status_code=400)
```

That view processes a submitted layer, validating the relevant forms and saving new objects before emailing admins. When you call `tasks.send_email`, Celery communicates with RabbitMQ to create a task for sending the emails. Then, as it consumes tasks from the queue, it sends the emails. This allows you to redirect the user that submitted the layer appropriately (ex: to a "thank you" page) without waiting for the emails to send. If you sent them synchronously, then any error while sending (such as a defunct email address) would show up for the user as a 500 error. Sending them asynchronously, you could instead log the error or take corrective action without holding up the user.

## Conclusion

That's it! You've set up an asynchronous task queue in Django. I hope you found these instructions helpful!