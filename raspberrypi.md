# building product w/ raspberry pi

one of my recent side projects was to build a digital clock/scoreboard for sports.  i decided to use a raspberry pi model 3b connected
to an led monitor as the solution.  i had a lot of fun building this project, but there were lots of tradeoffs and choices so
here are my experiences for anyone interested in building similar devices.  by the way, the terms of the raspberry pi are extremely
product-friendly, i.e. you can sell products based on raspberry pi without paying any royalties/license fees, the raspberry pi
foundation just suggests that you mention their product on your pages.

## use case

for my use case, the clock/scoreboard needed to be an asynchronous service that could support multiple simultaneous client connections for
reads and writes.  i originally thought about creating a mobile application connected to a service in the cloud, but for my digital
clock/scoreboard, i needed millisecond accuracy, the lag over wan would be a non-starter, lan is tolerable.  second, for mobile applications,
a phone call or other distraction would disrupt the service, remember this needs to be connected to an led monitor, you wouldn't want
people to see your private sms or facebook messages popping up as notifications in the middle of a game.  and lastly, running a service
on a mobile phone is not the right place, how would other clients connect to a device that's often times wading through different ip
addresses or potentially using mobile networks?  on top of the service, i needed a desktop environment that could display a browser.

to control the clock/scoreboard service, i needed a mobile application, iphone and android being the obvious platforms to support so
the backend had to support both of these.  i developed a swift ios application initially and used an apple tv to airplay the
content, but that would require an apple tv device which adds costs to the solution and would not allow me to support android clients
in the future.  plus this would make the mobile application a backend endpoint and wouldn't allow multiple connections to interact
with it.

## hardware stack

i chose raspberry pi because i had heard a lot of mention about it, it's quite affordable and performance is decent, plus i
always wanted to play around with one of these.  the raspberry pi is based on an arm v7 processor for low power consumption (quadcore),
includes hdmi output, 1g ram, onboard wifi router, the ability to use microsd cards for storage, is quite expandable with 4k cameras, sensors,
etc, and the size and form factor are just about right, like the size of a non-4k apple tv device.  for my digital clock/scoreboard, this
could hang on the wall or ceiling so small, energy efficient were very important.

## software stack

in terms of operating systems, i've always liked using linux and the recommended system on raspberry pi is raspbian, this is a debian
based linux distribution, but its purpose was specifically to create a desktop environment for schools or those learning to use
computers, there were lots of applications like libre office that just weren't needed, the size required for install was roughly 4g
and it used a strange windowing system.

i ended up using ubuntu-mate which required 8g storage, but this seemed much more familiar to me, it was running 16.04 LTS, unity
as the window system and the menus and everything seemed fairly familiar.

i looked at ubuntu core, which was built for iot devices, but ubuntu core leverages docker which was just overkill for this
project, i'd need additional memory for the docker processes, and there was this whole thing about forcing you to create docker images
and put them up on some snap service which required registration that just turned me entirely off.  i never measured how much more
resources i needed to run docker, but with 1g ram, it just didn't seem necessary, especially since this would be a gui application
running on the device.  the distribution itself was quite minimalized, but docker was a non-starter.

i also looked at fedora atomic, which is similar to ubuntu core--requirements for docker and k8.  microsoft has an iot os which is
based on windows, but this requires a complicated license scheme so that was quickly dismissed as a possibility.

i had learned golang around the time i started this project and decided to create a websocket server to manage the clock/scoreboard, this
would allow clients to update data asynchronously.  one really nice thing about golang is that the output is a binary file, so i didn't
really need to install a golang compiler onto the device which would save me space and complexity (think upgrades of golang, having to have
source code on the device itself, etc).  all persistent data is stored to an sqlite file, my data requirements are not large, probably a
few thousand to at most a million rows over 5 or so tables.

there needed to also be a web server to serve up static pages with embedded javascript.  this required a desktop environment to
automatically start a browser and open up the main page of the web application.


