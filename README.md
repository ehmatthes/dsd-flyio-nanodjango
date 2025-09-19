# dsd-flyio-nanodjango

A proof-of-concept django-simple-deploy plugin that deploys nanodjango projects to Fly.io.

Setup
---

If you don't have a sample nanodjango project to work with, you can clone the [nd_counter](https://github.com/ehmatthes/nd_counter) repo. This is a simple nanodjango project, taken from the [Getting started](https://docs.nanodjango.dev/en/latest/get_started/) docs.

You can clone this repository, run it once locally, and then deploy it.

```sh
$ git clone https://github.com/ehmatthes/nd_counter.git
$ cd nd_counter
nd_counter$ uv venv .venv
nd_counter$ source .venv/bin/activate
(.venv) nd_counter$ uv pip install -r requirements.txt
(.venv) nd_counter$ nanodjango run counter.py
```

Adding django-simple-deply
---

Installing this plugin will automatically install django-simple-deploy. Let's install the plugin, and freeze the requirements:

```sh
$ uv pip install dsd-flyio-nanodjango
 + django-simple-deploy==1.3.0
 + dsd-flyio-nanodjango==0.2.0
 (.venv) nd_counter$ uv pip freeze > requirements.txt
```

Now we'll add `django_simple_deploy` to the project script:

```python
from django.db import models
from nanodjango import Django

app = Django(
    EXTRA_APPS=["django_simple_deploy"],
)
...
```

Let's commit these changes:

```sh
(.venv) nd_counter$ git add .
(.venv) nd_counter$ git commit -m "Added django-simple-deploy and plugin."
```

Configuration-only deployment
---

In the configuration-only approach, you make the necessary resources on the remote server, review changes, commit them, and then push the project to the hosting platform.

Make an empty project on Fly.io that we can deploy to:

```sh
(.venv) nd_counter$ fly apps create --generate-name
New app created: nameless-bird-5390
```

Now call `deploy`, which will configure for deployment to Fly:

```sh
(.venv) nd_counter$ nanodjango manage counter.py deploy
...
Deployment target: Fly.io
  Using plugin: dsd_flyio_nanodjango`
...
--- Your project is now configured for deployment on Fly.io ---
...
```

Inspect the changes, and commit them:

```sh
(.venv) nd_counter$ git status
On branch main
Changes not staged for commit:
	modified:   .gitignore
Untracked files:
	.dockerignore
	Dockerfile
	fly.toml
(.venv) nd_counter$ git add .
(.venv) nd_counter$ git commit -am "Configured for deployment to Fly."
```

django-simple-deploy wrote all the changes necessary to serve the project on Fly.io. Now make the actual push to Fly:

```sh
(.venv) nd_counter$ fly deploy
...
```

Once the push finishes, you can open the deployed version of your project:

```sh
(.venv) nd_counter$ fly apps open
opening https://nameless-bird-5390.fly.dev/ ...
```

The counter will increment, as it did locally. In the 0.2.0 release of this plugin, the count will jump around because we're using SQLite on an ephemeral machine. Later releases will configure a persistent database.

When you're satisfied it works, make sure to destroy the deployed app if you don't want to accrue charges:

```sh
(.venv) nd_counter$ fly apps destroy nameless-bird-5390
Destroying an app is not reversible.
? Destroy app nameless-bird-5390? Yes
Destroyed app nameless-bird-5390
```

Fully-automated deployment
---

The fully-automated approach creates any necessary remote resources, configures your project for deployment, commits changes, and pushes your project to the remote server.

This is done using the `--automate-all` flag:


```sh
(.venv) nd_counter$ nanodjango manage counter.py deploy --automate-all
...
Deployment target: Fly.io
  Using plugin: dsd_flyio_nanodjango`
...
--- Your project should now be deployed on Fly.io ---

It should have opened up in a new browser tab. If you see a
  "server not available" message, wait a minute or two and
  refresh the tab. It sometimes takes a few minutes for the
  server to be ready.
- You can also visit your project at https://billowing-leaf-3573.fly.dev/
...
```

When you're satisfied it works, make sure to destroy the deployed app if you don't want to accrue charges:

```sh
(.venv) nd_counter$ fly apps destroy nameless-bird-5390
Destroying an app is not reversible.
? Destroy app nameless-bird-5390? Yes
Destroyed app nameless-bird-5390
```











