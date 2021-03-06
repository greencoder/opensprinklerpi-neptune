ospi-cli
======================

A command-line interface to run individual stations on your OpenSprinkler Pi for a set number of minutes.

**Disclaimer: This software is very alpha. Don't blame me if your yard is turned into a lake. There are a few built-in safeguards, but please use at your own risk and test thoroughly before you go on a month-long vacation. This software is running at my house, but your mileage may vary.**

###Usage###

    $ python opensprinkler.py --station [STATION_NUMBER] --minutes [MINUTES_TO_RUN]

Optionally, you can add the `--debug` flag if you want log messages to be printed to the console.

###Safety Mechanisms###

When `opensprinkler.py` runs, a PID file is created in the project directory. When the station finishes, this PID file is automatically removed. If something goes wrong, the file won't be removed, and you look for this file as a safety mechanism to prevent runaway jobs:

    $ sudo python utilities/check_pids.py

Currently, you have to run this utility with `sudo` so that you can turn off any stations that might be still running. I don't like this, and it will be cleaned up in the future.

*Note: Currently, the max watering time for a station is set to 30 minutes, so this script checks for any pid files that are older than 35 minutes. You can change this to suit your needs.*

###Adding Delay to Prevent Station Operation###

You can prevent jobs from running by placing a file named `DELAY` in the project directory. If present, the job will abort.

There is a file in utilities that looks at the body of this file to determine when it should be cleared. The format is YYYY-MM-DD HH:MM.

    $ python utilities/check_delay.py

If the file is found and the time in the file is older than the current time, the `DELAY` file is removed so that normal operations can resume.

As a convenience, there is a utility program to automatically create the file in the proper format:

    $ python utilities/set_delay.py --hours [NUMBER_OF_HOURS_TO_DELAY]

*Note: This would be a good mechanism to use if you have any programs that check the weather; you could write logic to figure out how long to set a delay based on the forecast.*

###Cron Scheduling###

The main purpose of this limited program is to allow you to schedule your stations by cron. Here's an example:

    # Run every Monday, Wednesday, and Friday at 5:00am
    00 05 * * 1,3,5 sudo /usr/bin/python /path/to/opensprinkler.py --station 1 --minutes 15

Remember, this is cron, so make sure to use full paths to your files. You must use `sudo` since Python needs to access the GPIO pins. (I don't like it either but it's just the way it works.)

###Known Limitations###

This program doesn't currently handle overlapping programs very well. It does work, but it's not elegant. When a station finishes, the program turns off all zones (I don't yet know how to read the current status of individual zones) so you may have a situation where your overlapping job mysteriously stops. One solution to this would be to look for the presence of a pid file and prevent the second operation from starting.

Once I get more comfortable with the hardware, I'm going to write a long-running service that acts as a job runner to gracefully handle overlapping requests. This will pave the way for a web-based interface.

###Feedback###

I welcome any feedback or advice you are willing to offer. You can use the issues tab in Github or reach out to me at snewman18 [at gmail dot com].

###License###

This project is distributed under the MIT license. Do whatever you want with it, but please provide attribution back and don't hold me liable.
