===============================================================================
               OMNIROUTE DOCKER SETUP & MANAGEMENT MANUAL
===============================================================================

-------------------------------------------------------------------------------
PART 1: TERMINAL SETUP & PREPARATION
-------------------------------------------------------------------------------

1. WHAT TERMINAL TO USE:
   - Primary Terminal: Ubuntu (WSL) terminal.
   - How to open it: 
     1. Press the Windows Key on your keyboard.
     2. Type "Ubuntu" (or "WSL").
     3. Click on the Ubuntu app icon to open the Linux command line.

2. DOCKER DESKTOP INTEGRATION:
   - Purpose: Allows your Linux terminal (Ubuntu) to communicate directly with 
     the Docker Desktop engine running on Windows.
   - How to enable it:
     1. Open the Docker Desktop app in Windows.
     2. Click the Settings Gear Icon (top-right).
     3. Go to Resources -> WSL Integration.
     4. Enable integration with "Ubuntu" and click "Apply & restart".


-------------------------------------------------------------------------------
PART 2: STEP-BY-STEP COMMAND MANUAL
-------------------------------------------------------------------------------

STEP 1: Remove Conflicting Packages
Terminal: Ubuntu (WSL)

Commands:
  sudo service docker stop
  sudo apt remove -y docker.io

Purpose:
  Removes standalone Linux Docker installation from inside Ubuntu so it does not
  conflict with Docker Desktop over system ports.


STEP 2: Create a Blueprint Working Directory
Terminal: Ubuntu (WSL)

Commands:
  mkdir ~/omniroute-docker
  cd ~/omniroute-docker

Purpose:
  Creates a dedicated folder named "omniroute-docker" in your home directory 
  and moves your command prompt inside it.


STEP 3: Create and Edit the Dockerfile
Terminal: Ubuntu (WSL)

Command:
  nano Dockerfile

File Contents to paste into Dockerfile:
-------------------------------------------------------------------------------
FROM node:22-bookworm-slim

RUN apt-get update && apt-get install -y dumb-init procps && rm -rf /var/lib/apt/lists/*

RUN npm install -g omniroute

WORKDIR /app

EXPOSE 20128

ENV OMNIROUTE_SERVER_HOST=0.0.0.0

ENTRYPOINT ["dumb-init", "--"]

CMD ["omniroute", "serve", "--no-open", "--log", "--no-recovery"]
-------------------------------------------------------------------------------

Explanation of Dockerfile instructions:
  - FROM node:22-bookworm-slim
    Downloads a base Debian Linux image with Node.js 22 pre-installed.
  - RUN apt-get update && apt-get install -y dumb-init procps ...
    Installs process management tools inside the image to manage ports safely.
  - RUN npm install -g omniroute
    Installs OmniRoute globally INSIDE the isolated container environment.
  - WORKDIR /app
    Sets /app as the default working directory inside the container.
  - EXPOSE 20128
    Tells Docker that the container listens on port 20128.
  - ENV OMNIROUTE_SERVER_HOST=0.0.0.0
    Sets an environment variable so OmniRoute accepts outside connections.
  - ENTRYPOINT ["dumb-init", "--"]
    Uses dumb-init to safely manage process signals and container execution.
  - CMD ["omniroute", "serve", "--no-open", "--log", "--no-recovery"]
    The default command that launches OmniRoute server in background mode.

(To save in nano: Press Ctrl + O, hit Enter, then press Ctrl + X)


STEP 4: Build the Docker Image
Terminal: Ubuntu (WSL)

Command:
  docker build -t omniroute-app .

Purpose:
  Reads the Dockerfile in the current directory (.) and compiles an image blueprint
  named "omniroute-app".


STEP 5: Run the Docker Container
Terminal: Ubuntu (WSL) or Windows PowerShell

Command:
  docker run -d -p 20128:20128 --name my-omniroute omniroute-app

Purpose:
  - `-d`: Runs the container silently in background mode.
  - `-p 20128:20128`: Maps port 20128 on Windows/WSL to port 20128 in the container.
  - `--name my-omniroute`: Names the running instance "my-omniroute".
  - `omniroute-app`: The image blueprint used to start the container.


STEP 6: Verify and Access the Application
Tools: Web Browser & Terminal

Commands:
  docker ps             # Checks if the container is running active
  docker logs my-omniroute # Displays live logs printed by OmniRoute

Web Access:
  Open Chrome/Edge and go to: http://localhost:20128


-------------------------------------------------------------------------------
PART 3: MANAGING YOUR CONTAINER & IMAGES DAILY
-------------------------------------------------------------------------------

CONCEPT CHECKS:
  - Container: The running application instance.
  - Image: The blueprint/installer saved on disk.
  * Note: You must stop and remove a container before deleting its image.

QUICK REFERENCE COMMAND MATRIX:

+----------------------+-----------------------------------------------------------+-------------------------------------------+
| Action               | Terminal Command (WSL/PowerShell)  | Docker Desktop GUI Method                                        |
+----------------------+-----------------------------------------------------------+-------------------------------------------+
| Stop Server          | docker stop my-omniroute                                  | Click Stop (Square) icon next to app      |
| Start Server         | docker start my-omniroute                                 | Click Play (Triangle) icon next to app    |
| Delete Container     | docker rm -f my-omniroute                                 | Click Trash Can icon in Containers tab    |
| Delete Image (Files) | docker rmi omniroute-app                                  | Go to Images tab and click Trash Can icon |
| Open CLI in Container| docker exec -it my-omniroute bash                         | Click container -> navigate to Exec tab   |
| To Reset Password    | docker exec -it my-omniroute npx omniroute reset-password | Click container -> navigate to Exec tab   |
+----------------------+-----------------------------------------------------------+-------------------------------------------+


COMPLETE REMOVAL / CLEAN WIPE COMMANDS:
Run these commands in order if you want to completely erase OmniRoute:

  1. docker rm -f my-omniroute   # Stops and removes container
  2. docker rmi omniroute-app    # Deletes image from disk
===============================================================================