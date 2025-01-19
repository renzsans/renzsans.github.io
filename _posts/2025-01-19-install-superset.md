---
title: "Install Apache Superset"
date: 2025-01-19
categories: [Apache Superset]
tags: [Analytics, Visualization, Software]
---

# Running Apache Superset as Python Virtual Environment on Windows

Sometimes even if documentation is available, software installation takes a bit more work than what the documentation provides. It is not a clear-cut sequential approach in installing software. In this case, I found SuperSet installation a bit more involved especially when running on Windows as a python virtual environment. 

SuperSet is an open source Data Analytics and Visualization tool by Apache. Anyone can certainly use this software to display and analyze their own data with some flair. In this article, I will try to explain a bit more step-by-step SuperSet installation as a python venv on a Windows machine. Hopefully, it will help anyone understand and follow from start to finish.

Installing SuperSet as python venv can be found in its own wesite here. https://superset.apache.org/docs/installation/pypi#python-virtual-environment

However, lets add more details from a start to finish approach.

## Install Python Software

For this, lets install python 3.11 on your Windows machine. Using Windows, I enjoy installing software as chocolatey packages since its much easier to manage all my software. If you already have python 3.6+, you can skip this step. 

Lets install chocolatey from this website https://chocolatey.org/install. Just copy the CLI command and run it in your powershell.
```
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
```

Now, install python 3.11 with this command.

choco install python --version=3.11.0

Your python should now be up and running.

## Create a Python Virtual Environment

First, install the virtualenv package.
```python
pip install virtualenv
```

Then, `cd` to a folder you want to work on and create the virtual environment. I will call mine `superset`.
```python
python -m venv superset
```

To activate the python virtual environment on Windows, run the command.
```batch
.\superset\Scripts\activate
```

You should see the venv name prefix in your CLI like below.
```
(superset) PS C:\Development\Superset>
```
![CLI View](/assets/img/posts/2025-01-19-install-superset/superset-activate-cli.png){: width="600"}

> **_Remember_**: You are inside the venv, if you see your venv name prefix in your CLI.
{: .prompt-tip }

> **_Note_**: You can deactivate the virtual environment by simply running `deactivate` inside the venv.
{: .prompt-tip }


## Install Apache SuperSet

Inside your venv, install the superset package. You have venv activated if you see the venv name prefix in your CLI, if not see previous step.
```
pip install apache-superset
```

Once installed, you will need to defined some mandatory configurations. I am assuming you are on a Windows machine as stated in the intro, therefore, we need to manually edit some files to include required configurations.

Assuming your venv is named `superset`, under the folder `\superset\Scripts` you will see multiple `activate` named files with differing file types. Open each one using a text editor and add the following lines after the last keyword command *(mostly towards the bottom of the page)*. The secret key can be any string you want but be consistent.

For `activate` bash file *(this is mostly for Linux but lets update it anyway)*:
```
export SUPERSET_SECRET_KEY=YOUR-SECRET-KEY
export FLASK_APP=superset
```

For `activate.bat` batch file:
```
set SUPERSET_SECRET_KEY=YOUR-SECRET-KEY
set FLASK_APP=superset
```

For `Activate.ps1` powershell file:
```
$Env:SUPERSET_SECRET_KEY = "YOUR-SECRET-KEY"
$Env:FLASK_APP = "superset"
```

Reactivate you venv to make sure the following environment variables are set. Run `deactivate` if inside venv, then `.\superset\Scripts\activate` to activate again.


## Run SuperSet with Examples

Inside your venv with superset installed, initialize a sample sqlite database with command below.
```python
superset db upgrade
```

Setup database as admin.
```python
# Create an admin user in your metadata database 
# (use `admin` as username to be able to load the examples)
superset fab create-admin

# Load some data to play with
superset load_examples

# Create default roles and permissions
superset init

# To start a development web server on port 8088, use -p to bind to another port
superset run -p 8088 --with-threads --reload --debugger
```
> **_Note_**: You can remove `--debugger` to save some processing time

If everything worked, you should now have Superset running with sample data. Navigate to hostname:port in your local browser *(e.g. locally by default at localhost:8088)* and login using the username and password you created.

![Login page](/assets/img/posts/2025-01-19-install-superset/superset-login-page.png){: width="600"}

![With examples](/assets/img/posts/2025-01-19-install-superset/superset-with-examples.png){: width="600"}