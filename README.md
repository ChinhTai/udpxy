#
# Copyright 2008-2012 Pavel V. Cherenkov (pcherenkov@gmail.com)
#
#  This file is part of udpxy.
#
#  udpxy is free software: you can redistribute it and/or modify
#  it under the terms of the GNU General Public License as published by
#  the Free Software Foundation, either version 3 of the License, or
#  (at your option) any later version.
#
#  udpxy is distributed in the hope that it will be useful,
#  but WITHOUT ANY WARRANTY; without even the implied warranty of
#  MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
#  GNU General Public License for more details.
#
#  You should have received a copy of the GNU General Public License
#  along with udpxy.  If not, see <http://www.gnu.org/licenses/>.
#

Summary
--------------

udpxy is a UDP-to-HTTP multicast traffic relay daemon:
it forwards UDP traffic from a given multicast subscription
to the requesting HTTP client.

udpxy is released under GPL v.3

Project status
--------------

udpxy has not been extended or supported for 4+ years, having been
replaced by Gigapxy - a superior enterprise-oriented product.
Please see more info at <http://gigapxy.com>, thank you.


Building and installing
--------------

Untar the *.tgz source distribution into a directory of your choice by
running
        tar -xzvf udpxy.X.Y-ZZ.tgz
    or
        gzip -dc udpxy.X.Y-ZZ.tgz | tar -xvf -

Make sure GNU make and gcc are available (gcc 3.x and up should work, lower
versions are not guaranteed to build the source correctly); for compilers
other than gcc alterations to Makefile might be needed.

Running 'make' without a target will build the 'release' version of
udpxy (no asserts, no debug symbols, verbose mode on).

Other make targets are:
    debug (asserts, debug symbols, verbose mode on);
    lean  (no asserts, no debug symbols, verbose mode off);
    rdebug (same as 'release' but with debug symbols);
    ldebug (same as 'lean' but with debug symbols);

Once the make has succeeded, the udpxy executable file could be
copied to a location of one's choice and run from there - no additional
installation steps are required.

udpxy can be started with a number of configuration parameters,
such as listening address/port, multicast interface name, etc.
A brief usage summary is provided when udpxy is invoked from command line
without parameters.

HTTP commands
--------------

udpxy responds to HTTP (GET) commands to receive data from
a dedicated multicast group and forward it to the initiating (HTTP)
connection.

The command to relay traffic is in the format as below:

http://address:port/cmd/mgroup_address[SEP]mgroup_port/

[SEP] ::= :|%|~|+|-|^
i.e:
    http://ip:port/cmd/mgroup_address:mgroup_port/
    http://ip:port/cmd/mgroup_address%mgroup_port/
    http://ip:port/cmd/mgroup_address~mgroup_port/
    ......
    http://ip:port/cmd/mgroup_address^mgroup_port/

are acceptable and should all work in the same manner.

cmd ::= udp | rtp

where ip and port match the listening address/port combination of udpxy,
and mgroup_address:mgroup_port identify the multicast group to subscribe to.

Using 'udp' command will instruct udpxy to probe for known types of payload
(such as MPEG-TS and RTP over MPEG-TS); using 'rtp' makes udpxy assume RTP
over MPEG-TS payload, thus skipping the probes.

udpxy will start a 'client' process for each new relay request as long as
their number would not exceed a pre-set maximum (see usage summary).

udpxy also supports a few additional HTTP requests, such as:

http://address:port/status/  - to display basic daemon's statistics
http://address:port/restart/ - to close all active connections and restart

Payload types and handling
--------------

udpxy recognizes MPEG-TS and RTP (over MPEG-TS) payloads within relayed packets;
if udpxy encounters RTP payload it automatically 'translates' it to MPEG-TS so that
media players not recognizing RTP on TCP could still play back the stream.

So far, no translation is performed for other payload types.

Recording MPEG traffic
--------------
udpxy (in builds >0.33) includes functionality to record captured traffic as
raw MPEG-TS stream into a file. This functionality is enabled through udpxrec:
a bundled-in application that is linked together with udpxy (as one executable).

udpxrec is invoked by a symbolic link (named udpxrec) to the udpxy executable
(NB: do not rename udpxy executable).

udpxrec creates MPEG files encapsulating MPEG-TS segments; most media players
will *NOT* play such files; to make them playable the stream must be transcoded
to MPEG-PS; vlc knows how to do such transcoding, here is a command-line example:

vlc input-ts.mpg --sout="#std{access=file,mux=ps,dst=out-ps.mpg}"

The resulting PS file can be played back by most media players.

Portability
--------------
udpxy was written to run on 'POSIX-compliant' systems;
so far all builds have been tested to build and run on Linux 2.4, 2.6, 3.x (IA32, ARM)
and *some* (but not all) on HP-UX 11.11 (PA-RISC 1.1, 2.0w).

Build 12 of version 1.0 (Chipmunk) was ported to compile under FreeBSD 7.1
using GNU make 3.8; later builds have been tested to compile under later
versions of FreeBSD (up to 9.0);

Certain builds have been ported to build under cygwin; cygwin is
*NOT* considered to be a fully supported platform yet there is an
ongoing effort to make udpxy run on it as well.

Environment variables
--------------
udpxy utilizes the following environment variables to compliment its
command-line options; the variables are considered for the options that
most people would not need to change too often (or simply inconvenient
to use from the command line).

NB: If there is a command-line switch that would intersect in functionality
with an environment variable, the switch *always* has the higher priority.

UDPXY_RCV_TMOUT         - timeout (sec) on the inbound data stream (multicast), default=5;
UDPXY_DHOLD_TMOUT       - timeout (sec) to hold buffered data before sending/flushing to client(s), default=1;

UDPXY_SREAD_TMOUT       - timeout (sec) to read from the listening socked (handling HTTP requests), default=1;
UDPXY_SWRITE_TMOUT      - timeout (sec) to write to the listening socked (handling HTTP requests), default=1;
UDPXY_SSEL_TMOUT        - timeout (sec) to select(2) in server loop (unused if pselect(2) is employed), default=30;
UDPXY_LQ_BACKLOG        - size of the listener socket's backlog, default=16;
UDPXY_SRV_RLWMARK       - low watermaek on the receiving (m-cast) socket, default=0 (not set);
UDPXY_SSOCKBUF_NOSYNC   - do not sync inbound (UDP) socket's buffer size (with value set by -B), default=1 (sync);
UDPXY_DSOCKBUF_NOSYNC   - do not sync outbound (TCP) socket's buffer size (with value set by -B), default=1 (sync);

UDPXY_TCP_NODELAY       - disable Nagle algorithm on the newly accepted socket (faster channel switching), default=1;

UDPXY_HTTP200_FTR_FILE - append contents of the given file to the HTTP 200 response, default=none;
UDPXY_HTTP200_FTR_LN   - append the text (line) to the  HTTP 200 response, default=none;

UDPXY_ALLOW_PAUSES     - if blocked on a write, keep reading data until the buffer (-B size) is full, default=disabled;
UDPXY_PAUSE_MSEC       - allow only N milliseconds of reading data when blocked on a write.
UDPXY_CONTENT_TYPE     - specify custom Content-Type in HTTP responses.

--EOF--
