# Using GNU screen to run the servers in the background

## Introduction

GNU screen is a venerable program to multiplex and manage terminals that can be detached and reattached at will. There are numerous alternatives these days, feel free to use any that you like, but if you choose to use screen then here is a sample configuration.

## In a nutshell: using screen

Once you have screen installed using your distribution's package system, you can simply run the command `screen` at the command line. It will then open up a terminal again, but this time running within screen. By default, you can issue commands to screen by pressing `Ctrl-A` and then another letter. So, for example, if you type `Ctrl-A c` (which to be clear means first press the Control key along with the A key, then let go of both and press the C key), it will 'create' a new terminal window. Now you have two terminal windows, and you can switch back and forth using `Ctrl-A n` (next) and `Ctrl-A p` (previous). You can see a brief overview of the terminal windows using `Ctrl-A w` (windowlist) that will either pop-up at the bottom of your screen or show in your terminal's titlebar if it has one. There are many commands and we cannot go into them all, see the [GNU screen manual](https://www.gnu.org/software/screen/manual/) for more info, or hit `Ctrl-A ?` for a quick overview.

Most relevantly, when you are inside screen you can type `Ctrl-A d` to detach from screen and exit screen while the terminals and programs inside of it keep running. This is very useful for keeping long-running programs going on a remote server. Or, if you happen to lose your SSH connection to the server, the programs will continue running inside of screen, no problem. Either way, the next time you login to your server via SSH, you can restore your screen session with the command `screen -dr`.

## Starting the servers automatically


Open a file named `percept-screenrc` in your home directory and put the following contents into it:

    startup_message off
    defscrollback 5000
    utf8 on

    screen 0 bash -ilc '(cd percept-frontend; serve -s build)'
    screen 1 bash -ilc '(cd percept-backend; npm start)'
    screen 2 bash
    select 2

This assumes that you have downloaded and configured the [backend](backend.md) and [frontend](frontend.md) software, and placed them into the `percept-backend` and `percept-frontend` subdirectories of your home directory, respectively. If not, please adjust the paths in the above example before going further. It also assumes that you used `npm run build` on the frontend to build a compiled version that can be served using the `serve -s build` command. If you'd rather use `npm run dev` to run the debug version, then adjust the above example appropriately.

Now if you run `screen -c ~/percept-screenrc` then it will open a screen session and start both frontend and backend servers, and it will open a new terminal window with a plain bash prompt. If you wish to view the output of either server, simply switch to those windows using e.g., `Ctrl-A p`.

## Starting the servers automatically at reboot

There are several ways to do this but the easiest at this point is simply to use your crontab. This is a file that contains commands which run periodically. Conveniently, you can also define a command that occurs after reboot. Simply run the command `crontab -e`, which will open the crontab file in your favourite editor. Insert the following line into the file, at the very start of the file, save and close it:

    @reboot screen -d -m -c ~/percept-screenrc

Next time you reboot and login, there should be a screen session running already. You can simply reattach to it using `screen -dr` and check it out.

