# Instructions to recreate a local copy of the PostgreSQL databases in the VM on Coursera

## Motivation

I have a set of database tools that I'm comfortable using, so I wanted to set up a database environment for the SQL course that lets me use those tools. I'm not a huge fan of using pgAdmin unless I have specific use cases where it's a better choice -- typically when I need to take advantage of its import/export features.

I also ran into some network lag when using the Coursera VM.

I read some comments saying that the VM doesn't let you copy and paste between itself and your local machine. So that's another barrier.

In case you're interested, my general setup is to run an RDBMS in Docker and connect to it usually from Azure Data Studio (ADS). ADS is similar to VS Code but is geared for working with databases. I like that I can use a Vim plugin for it, and I do a lot of work in ADS and VS Code already. This also lets me pretty easily modify the Docker config files and test out a different RDBMS if I want to.

# Walkthrough

1. Launch the VM and make sure you can see the databases there.

Nothing fancy here. Just launch the VM from the Coursera page and wait for it to load in the browser.

2. Get Local Postgres-Specific Copies of the Databases

We need to get a copy of the Northwinds and ClassicModels databases that are already initialized in the VM. I did see some scripts for SQL Server and MySQL, but I wanted to match the RDBMS used for the class, so I wanted to get a solid set of scripts to recreate the databases in Postgres. I also thought it would be easier to pull copies of the existing databases instead of trying to convert between different RDBMS scripts.

Log into the database through pgAdmin on the VM and make sure that you know the username and password for an admin account. (Going through this again, this may not be needed, but I left it here anyways since it can be helpful to have a visual for the databases in case you need to troubleshoot anything from within pgAdmin. I always forget that the databases have uppercase letter in them and only remember once I look at them through pgAdmin.)

When you install Postgres, you get some command line tools. One of those is pg_dump. This dumps or extracts a database into a .sql file that you can then run from within another Postgres database.

Next we're going to do some steps from the Linux terminal. If you aren't familiar with Linux and need some help, ping me on Slack.

- Open the terminal in the VM
- Create a new directory to put the output scripts: `mkdir pg_scripts`
- Move into the new directory: `cd pg_scripts`
- Dump the Northwinds database: `pg_dump Northwinds > nw.sql`
- Dump the ClassicModels database: `pg_dump ClassicModels > cm.sql`

Note that the databases are in Pascal case, and the command is case sensitive.

Now we have .sql files that contain both DDL and DML to recreate the databases in a new instance of Postgres.

Next we'll move these files into a location where we can download them. In the Coursera web UI, when you select Lab Files in the upper right, you see a /home/developer/workspace directory. This exists on the VM in which we're accessing pgAdmin and using the terminal. So, we need to copy the sql files over to this location so that we can then download them from the Coursera web UI.

Back in the terminal in the VM. Make sure you're in the same directory as the sql scripts from above.

- Copy the Northwinds sql file to the download location: `cp nw.sql /home/developer/workspace/`
- Copy the ClassicModels sql file to the download location: `cp cm.sql /home/developer/workspace/`

Now click on Lab Files in the Coursera web UI. Click on the /home/developer/workspace directory. You should see nw.sql and cw.sql there. You can download these like you would Jupyter notebooks or data files or whatnot in other courses so that they're on your local machine.

At this point, you have two .sql files in pgSQL, a variant of SQL specifically for Postgres. If you decided to grab for instance a MySQL or SQL Server variant of SQL to load the data from, this is the version but for Postgres so you don't have to start translating the little differences in syntax.

3. Set Up Local RDBMS

My preferred solution is to use Docker to run Postgres locally. I'll document that here. But, it's totally fine if you prefer to install Postgres and pgAdmin locally on your computer. There's nothing fancy about that, so follow the official install instructions from the Postgres website(s). Once you have them running, you can then proceed to load the sql files into your Postgres instance. You don't need to worry about Docker if you're installing things directly on your computer.

If you do want to use Docker, I'll include the compose file in this same directory so you can snag that and modify it as you want.

For those who aren't familiar with Docker, it is software that runs containers. A container is sort of like the virtual macine we're using in Courera, but it's lighter-weight. If you aren't familiar with Docker or not interested in using it, I'd say skip this. If anyone really wants to use Docker or learn about it and get some experience with it, ping me on Slack for more info. I will say that I personally think Docker is an important piece of data science tech, even if it is more on the IT and developer side as opposed to maybe the analyst side.

Docker setup commands:

1. Make sure you have Docker and compose installed
2. Grab the compose file in this directory
3. Download the image and start up the container by running this in the same directory that has the compose file on your file system: `docker compose up` (if you want to see output in the terminal) or `docker compose up -d` (if you want to run in detached mode and not see output in the terminal)
   - Alternatively, you can use the Docker Desktop UI to start up the container. I've actually never set up containers with the GUI tool, but ping me on Slack if you want to go that route and need some help. I learned Docker using the terminal, so that's how I've kept using it since.
4. Connect to the Postgres instance from your IDE of choice and create the Northwinds and ClassicModels databases, either with the UI or with SQL: `create database Northwinds` and `create database ClassicModels`
   - Running the sql scripts later won't actually create the databases, so that's why we create them as their own step here.
5. Once the container is running, we need to copy the sql files into the file system inside the container: `docker cp nw.sql <container id>:/` and `docker cp cm.sql <container id>:/` (note that we're putting the sql file in the root of the file system inside the container)
   - You could add this step into the compose file if you want. I did it manually. I can update the compose file later if that'd be helpful and if anyone is actually going to do this in Docker.
6. Connect to bash in the container: `docker exec -it <container_id> bash`
7. Load the files into the Postgres instance. Make sure you're in the same spot in the container file system where you copied the files into. `psql Northwinds < nw.sql` and `psql ClassicModels < cm.sql`

Non-Docker setup:

You can run step 6 from the Docker setup just above from your local terminal if you're comfortable with that.

I'll walk through this assuming you're using pgAdmin. That's the typical IDE you'll use at first with Postgres if you don't have an alternative that you prefer.

I haven't tested this process, but I'm pretty sure this is right. Let me know if it isn't so I can correct this bit.

1. Launch pgAdmin
2. Connect to the local Postgres instance
3. Create the Northwinds and ClassicModels databases. See step 4 above for SQL if you want to do it that way. You can also use the GUI to create the databases.
4. Right-click on the database you want to load and select PSQL Tool
5. In the psql console that comes up, type in `\i <path to sql file>`
   - You can open up the location on your local file system with the sql files in File Explorer or Finder to get the full path. Copy and paste that here. For Windows, that will start with c: and with MacOS that'll start with /Users

# Resources

I'll put the docker-compose.yml that I used for creating the Postgres instance. It expects you to have a .env file in the same directory as the yml file. The .env file needs a PG_PWD key in it. Alternatively, you can remove ${PG_PWD} from the yml file and hard-code your password in there instead.

Let me know what other resources would be helpful to load into here.

I'm not technically in the SQL course yet. I'm planning to pay to upgrade to for-credit next session and do the final then. If anyone else knows or can ask a TA if it's okay to load the sql files into GitHub or here on Slack, that's an option too. I'll have to wait until January until I'm in the class to ask that myself.
