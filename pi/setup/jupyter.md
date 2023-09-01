# Install Jupyter Lab
Connect to your Raspberry PI using SSH. Install all the dependencies. We will use pip3 to install jupyter lab later.
```
$ sudo apt-get update
$ sudo apt-get install python3-pip
$ sudo pip3 install setuptools
$ sudo apt install libffi-dev
$ sudo pip3 install cffi
```
Install jupyter using pip3. If you are using virtualenv then follow the instructions from the jupyter documentation here.

```
$ pip3 install jupyterlab
```
Create a directory for your notebooks and start jupyter lab (optional)
```
$ mkdir notebooks
$ jupyter lab --notebook-dir=~/notebooks
```
Alternately you can also start the jupyterlab from any directory as below.
```
$ jupyter lab
```
This should will start the jupyterlab running on https://localhost:8888 on raspberry pi but its not accessible from the your local machine. Also once you close your SSH session, the jupyterlab instance will be terminated.

To resolve this issue, we need to start the jupyterlab as a service.

# Setup Jupyter lab as a service
Run below command to locate your jupyter lab binary:

```$ which jupyter-lab
/home/pi/.local/bin/jupyter-lab
```
Create a file /etc/systemd/system/jupyter.service
```
$ sudo nano /etc/systemd/system/jupyter.service
```
With below content. Make sure to replace the path to jupyter-lab binary which was returned by which jupyter-lab command if its different for your system.

```
[Unit]
Description=Jupyter Lab
[Service]
Type=simple
PIDFile=/run/jupyter.pid
ExecStart=/bin/bash -c "/home/pi/.local/bin/jupyter-lab --ip="0.0.0.0" --no-browser --notebook-dir=/home/pi/notebooks"
User=pi
Group=pi
WorkingDirectory=/home/pi/notebooks
Restart=always
RestartSec=10
[Install]
WantedBy=multi-user.target
```

## Enable the service to start it whenever the raspberry pi boots
```
$ sudo systemctl enable jupyter.service
$ sudo systemctl daemon-reload
```
Start the service:
```
$ sudo systemctl start jupyter.service
```
You can also stop the service using command:
```
$ sudo systemctl stop jupyter.service
```

Reboot the raspberry pi and make sure that the service is running by checking the service status
```
$ sudo systemctl status jupyter.service
jupyter.service - Jupyter Notebook
   Loaded: loaded (/etc/systemd/system/jupyter.service; enabled; vendor preset: enabled)
   Active: active (running) since Sun 2020-08-30 16:12:27 PDT; 2s ago
 Main PID: 4864 (jupyter-lab)
    Tasks: 1 (limit: 4915)
   CGroup: /system.slice/jupyter.service
           └─4864 /usr/bin/python3 /usr/local/bin/jupyter-lab --ip=0.0.0.0 --no-browser --notebook-dir=/home/pi/notebooks
```

The service should be up and running and you can now access it from your local machine by going to http://<rpi ip address>:8888


However, this will be also accessible from anywhere on internet (depending on your router settings, firewall etc). We can secure it using one of the below three methods

### Add password
Check to see if you have a notebook configuration file,jupyter_notebook_config.py. The default location for this file is your Jupyter folder located in your home directory:

```
/home/pi/.jupyter/jupyter_notebook_config.py
```
If you don’t already have a Jupyter folder, or if your Jupyter folder doesn’t contain a notebook configuration file, run the following command:

```
$ jupyter notebook --generate-config
Writing default config to: /home/pi/.jupyter/jupyter_notebook_config.py
$ jupyter notebook password
Enter password:  ****
Verify password: ****
[NotebookPasswordApp] Wrote hashed password to /Users/you/.jupyter/jupyter_notebook_config.json
```

This can be used to reset a lost password; or if you believe your credentials have been leaked and desire to change your password. Changing your password will invalidate all logged-in sessions after a server restart.