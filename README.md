# Distributed Fuzzing for afl

This project is a quick and very dirty attempt (it's written in a few evenings) to combine the computing power of many machines that are outside my control for fuzzing with the great tool "afl". It works by a server that provides the projects to fuzz and a client running on the machines that downloads 1 or more projects (depending on the amount of CPU cores), downloads the queue and starts fuzzing. The new generated testcases are uploaded to the queue to sync to other clients and the crashed and hangs (after deduplication) are uploaded so they can be manually analyzed. The client will also check on a regular base if the project is modified. It will upgrade and restart the project when a update is detected. The client itself is not (yet) self updating.

The server is able to support without any problem tens of clients, however with very big queues the sync algorithm will become a bottleneck. The calculation of the hashes of the queue and other folders is (currently) not cached.

**Warning: Use this only with (semi) trusted clients and never connect to a untrusted server. The client will download and run arbitrary executable code from the server and nothing will prevent a client to submit a malicious sample to the queue that actually exploits a vulnerability.**

## Todo

- Make the client self-updating
- Make it possible to have a project run multiple times on the same client (a quick workaround is to copy the project multiple times and use $aggregateMap to combine the queues)
- Upload more metadata so it can plot fancy graphs based on all clients 
- Rewrite the server to something decent or at least do a serious clean up. (It's also not very secure atm, but you should only use this in a trusted environment as malicious clients are a much bigger risk)
- Add some kind of admin panel

## Install @ client

sudo apt-get install python python-dev python-pip tmux screen  
sudo pip install requests psutil tmuxp  
mkdir ./fuzzing  
sudo mount -t tmpfs -o size=2G tmpfs ./fuzzing  
cd ./fuzzing  
wget https://www.example.com/disfuzz.py  
chmod +x disfuzz.py

## Install @ server

- Upload the provided PHP scripts to a webserver
- Give write access to the data folder
- Add your project to the baseline folder (an example is provided)
- Modify the config to include your project

## Usage: Automated fuzzing

screen ./disfuzz.py auto

Close now your terminal window or use ^a + d to detach!

## Usage: Manual fuzzing

./disfuzz.py list  
./disfuzz.py init <projectname>  
./disfuzz.py start <projectname> [-m]  
screen ./disfuzz.py monitor

Close now your terminal window or use ^a + d to detach!

Note: The first but ONLY the first client must start itself as master instance  
Note: Make sure that the monitor system is always running! Otherwise your results are not uploaded and the results of other instances not imported.

## Security

- Provide the server component over HTTPS when its used over an untrusted network as otherwise an attacker can VERY easily get RCE on the nodes by a MITM attack as the client is self updating and will execute update.sh as part of the update process.
- Client will download and execute code coming from the server, so only connect to known and trusted servers
- The server will accept any submitted testcase which will be spread to all connected clients. Malicious clients could use this to exploit a vulnerability and get RCE on other clients or the machine of the administrator testing the submitted crash case.
