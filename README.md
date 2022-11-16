# Docker Installation Guide with OpenVas
This guide will go over the installation of Docker Desktop Executable for Windows 10 as well as the OpenVas vulnerability scanner. For this demonstration, I chose to use Ubuntu for Windows which you can acquire through the Microsoft Store and Docker Desktop to pair the kernel and pull containers. Also with the use of Docker Desktop and OpenVas from github, a .yml codeblock wasn't needed since the app configures it for you but for this installation it will be provided.

The provided links are for the download of Docker Desktop as well as Windows Terminal for the use of Ubuntu. Instructions for installation are included within these links.
_________________________
Docker Desktop: https://docs.docker.com/desktop/install/windows-install/

Windows Terminal: https://learn.microsoft.com/en-us/windows/terminal/install
_________________________

1. Open the Microsoft Store and search for 'Ubuntu', install it and then proceed to installing the Windows Terminal executable which will then pull open an empty terminal shell once finished installation. Upon opening, you will need to click the downward arrow located above the window header and select the settings option. In the profiles tab, select 'Ubuntu' and open a new tab to bring up the Linux bash terminal.

2. Run the Docker Desktop executable and follow the installation process accordingly. Afterwards, open Windows Powershell in the search bar and run it as an administrator for you will need privileges in order to swap from WSL 1 to WSL 2 however you first need to install WSL by typing in the command `wsl --install -d Ubuntu` in your powershell commandline. Once it finishes, you will then enter `wsl -l -v` to view the version of WSL currently running which should come back as 1. If so, you will need to change its version to 2 by entering the command `wsl --set-version Ubuntu 2` so it can run virtual memory off on the CPU allowing docker to function.

3. Run Docker Desktop once WSL 2 is configured and once signed in with an account, you want to then hit the gear shaped icon next to your name to open the docker settings. Select the **Resources** tab and in the drop down select **WSL Integration** and under "Enable integration with additional distros", turn on the Ubuntu distro. Afterwards, restart and refresh docker for it to take effect and test it by pulling dockers hello world image inside of your 'Windows Terminal' by typing the command `docker run hello-world` which will then pull the image and echo back a welcome introduction to docker in the terminal and a container image on the desktop app will now have an image under the name 'cranky_poincare' to show it was pulled successfully.

4. To install OpenVas, run the command `docker run -d -p 443:443 --name openvas mikesplain/openvas` in your windows terminal which will pull the image container from GitHub for installation. Once it reads back the installation was successful, you then need to set up your docker-compose file through the following code block:
```python
    version: '3'
services:
  # This Nginx will be the first to start, and it will serve the redirect as well as ACME verification
  nginx:
      image: nginx:alpine
      restart: always
      hostname: nginx
      ports:
        - "80:80"
      links:
        - openvas
      volumes:
        - ./conf/nginx.conf:/etc/nginx/nginx.conf:ro
        - ./data/letsencrypt:/etc/letsencrypt
        - ./data/letsencrypt-www:/tmp/letsencrypt
  # This Nginx requires the certificates to exist, otherwise will fail
  nginx_ssl:
      image: nginx:alpine
      restart: always
      hostname: nginx_ssl
      ports:
        - "443:443"
      links:
        - openvas
        - letsencrypt
      volumes:
        - ./conf/nginx_ssl.conf:/etc/nginx/nginx.conf:ro
        - ./data/letsencrypt:/etc/letsencrypt
        - ./data/letsencrypt-www:/tmp/letsencrypt
  letsencrypt:
      restart: always
      image: kvaps/letsencrypt-webroot
      volumes:
        - ./data/letsencrypt:/etc/letsencrypt
        - ./data/letsencrypt-www:/tmp/letsencrypt
      links:
        - nginx
      environment:
        DOMAINS: https://127.0.0.1
        EMAIL: <Insert Email>
        WEBROOT_PATH: /tmp/letsencrypt
        EXP_LIMIT: 30
        CHECK_FREQ: 30
  openvas:
      restart: always
      image: mikesplain/openvas
      hostname: openvas
      expose:
        - "443"
      volumes:
        - "./data/openvas:/var/lib/openvas/mgr/"
      environment:
        # CHANGE THIS !
        OV_PASSWORD: admin
      labels:
         deck-chores.dump.command: sh -c "greenbone-nvt-sync; openvasmd --rebuild --progress"
         deck-chores.dump.interval: daily
  # Daily updates to openvas
  cron:
      restart: always
      image: funkyfuture/deck-chores
      volumes:
        - "/var/run/docker.sock:/var/run/docker.sock"

```
Touch a file in the directory of your choice and edit the password and username using NANO to convert the text into a .yml file named *docker-compose.yml* which you will then run using the command `docker-compose up -d`.

5. Afterwards, you then want to make sure in the Docker Desktop app that **openvas** appears in the containers list and is running. If it is not running, you then need to hit the play button to start it and afterwards open up your browser (either Firefox or Google Chrome) and type in the url "https://127.0.0.1" and enter, it will then open up a page to the Greenhorn Security GUI. From there you can then login using either your own username and password that you picked in the .yml or by logging in with the default admin credentials. Once in, you can then run a vulnerability scan through the task management system which can allow you to proceed through scanning IPs of your choice.
   
This concludes the installation of Docker Desktop and OpenVas as well as the prerequisites to enabiling the features for a Windows OS. The following lines are photos containing the Docker App itself as well as OpenVas running on the GUI.

![Screenshot](https://raw.github.com/JGRUNE97/docker-readme-openvas/ProofconceptDocker1.jpg "OpenVas Vulnerability Scan")
![Screenshot](docker-readme-openvas/ProofconceptDocker2.jpg "OpenVas Vulnerability Scan")
