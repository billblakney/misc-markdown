# Installing Trilium on a Synology NAS
The goal of this guide is to configure and run a Trilium server on a Synology NAS, and configure it so that it can be accessed by the local network, by running Trilium from a web browser on the local network, or from an installed Trilium desktop client.

These notes are based on the following article:  [How To Install Trilium On Your Synology NAS With Docker ](https://neellik.com/how-to-install-trilium-on-your-synology-nas-with-docker/)

## Setup a Docker Task
Login to your Synology NAS and do the following:
1. In **File Station**, create a new `trilium` folder under `docker`, i.e. `/docker/trilium`.
2. Go to **Control Panel>Task Scheduler**. Select **Create**, then **Scheduled Task -> User-defined script**. This brings up the **Create task** dialog.
2a. In the **General** tab, in the **Task** field, enter _Trilium_. From the **User** list, select ***root***. Uncheck the **Enabled** option.
2b. In the **Schedule** tab, select ***Run on the following date***, set the date to today's date, and select ***Do not repeat***.
2c. In the ***Task Settings*** tab, optionally setup notifications to the email address of your choice.
2d. In the ***Run command*** section, enter the following script. You will need to change the mapped port value, 8482, if that port is not available on your NAS, or you just prefer to use a different port.
```sh
        docker run -d \
            --name trilium \
            -p 8482:8080 \
            -v /volume1/docker/trilium:/home/node/trilium-data \
            zadam/trilium:latest
```

3. Click **OK**, and then click **OK** when the warning message pops up.

## Run the Docker Task
On completion of the previous step, the **Create task** dialog closes, and you should see _Trilium_ in the list of tasks (while still in the **Task Scheduler** section of **Control Panel**.
1. Click on the _Trilium_ row, and press the **Run** menu item. Click **Yes** when the confirmation dialog pops up.
2. Go to the **Docker** application and verify that _Trilium_ is listed as a container. It should be in the _Running_ state.
## Setup a Reverse Proxy
At this point a Trilium container is up and running on your NAS, but you won't be able to access it over the local network. To enable local network access, you'll need to configure a reverse proxy.
1. Go to **Control Panel>Login Portal>Advanced**. Click the **Reverse Proxy** button and then the **Create** button. 
2. In the **Reverse Proxy Rules** dialog that pops up:
2.a In the **General** section, enter _Trilium_ for **Reverse Proxy Name**.
2.b In the **Source** subsection, select _HTTP_ for **Protocol**, enter _<NAS_hostname>_ for **Hostname**, and _<external_port>_, e.g. _8000_, which is the port you will use to access Trilium over the network, for **Port**.
2.c In the **Destination** subsection, select _HTTP_ for **Protocol**, _<NAS_IP_address>_ for **Hostname**, and _<the_port_from_step_2d>_, i.e. _8482_ for **Port**.
2.d Click **Save**, the **Close**.
## Connect Via Browser on Local Network
1. From a browser on your local network (i.e. the local network where your NAS is located), direct browser to _http://<nas_hostname>:<external_port>_, where _<nas_hostname>_ and _<external_port>_ are the values entered above, while setting up a reverse proxy. This should take you to a page titled _Trilium Notes setup_.
2. Select **I'm a new user, and I want to create new Trilium document for my notes**. After initializing the database on your server, you will be prompted to set a password, and then redirected to a page where you can login.
3. Login and use Trilium.

Once you've setup the server with that first login, on subsequent logins to Trilium you will be taken directly to the login page. Once you login, you should see any new notes, etc. that you made during the first login. Every login session from your browser will be sharing the data maintained by the Trilium server on your NAS.
## Connect Via the Desktop Trilium App on Local Network
Once you've installed the Trilium desktop app (not covered here), you can run it and connect to the existing database that you created when you connected to the server for the first time from a browser, as described in the previous section.
1. Start the Trilium desktop app.
2. Select **I have server instance already, and I want to set up sync with it** and click **Next**.
3. On the _Trilium Notes setup_ page:
3.a For **Trilium server address**, enter _http://<NAS_hostname>:<external_port>_.
3.b For **Password**, enter the password you chose when logging in from the browser.
Once the syncing has complete, you should see the same Trilium data that you saved in your previous browser sessions.

Note that any changes you make from a web browser or desktop app will be synced across all instances, as they are all using the same database maintained in the Trilium container running on your NAS.

## Appendix: Reinstalling Trilium on your NAS.
If for some reason you want to wipe out your existing Trilium server on your NAS, and start with a fresh database, you'll need to stop the existing Trilium container, delete the database, and restart the Trilium task.
1. In **Docker>Container**, select the _trilium_ item by clicking on it, then execute the **Stop** action.
2. Next, execute the **Delete** action.
3. In **File Station**, remove the `trilium` folder under `docker`, i.e. `/docker/trilium`.
4. Go to **Task Scheduler** and start the task that was entered when originally setting up the Trilium container.
## Appendix: Notes on the Trilium desktop app configuration in Linux.
The Trilium desktop app in Linux normally stores files in the _/home/<user>/.local/share/trilium_data_ directory. If for some reason you want to discard that data and instead use the data on your NAS, you can remove the directory and then connect your Trilium desktop app the the NAS as described above.

