Contents:


# Description #
There is a nice example on how to write a simple but already powerful file sharing webserver for a local network: http://fragments.turtlemeat.com/pythonwebserver.php .

In particular, this is very useful if one wants to quickly exchange files with a mobile phone over WiFi (when there is no Internet connection or the traffic counts, etc).

This page contains a few improvements (hopefully) to the original code, where the most important is that we now upload the file under it's original filename.

It could also be useful as a working example of how to use Python <tt>cgi.FieldStorage</tt> component with a non-CGI Python webserver like SimpleHTTPServer .

<a href='Hidden comment: 
This mini-project consists of four small parts:
'></a>

This mini-project consists of three small parts:

  * this intro
  * a code snippet that you might want to take home )
  * links to some other related pages in this wiki and outside.

<a href='Hidden comment: 

...

The fourth part is a working piece of code in the Mercurial repository that is *not* recommended to take home - it is something made very quick and very dirty (and quite messy), and I was unwilling to waste any additional time on it since it worked for me. It is strongly advised to start with the code snippet and make something better
'></a>

<a href='Hidden comment: 
Ж:-)
'></a>

# Code Snippet #

```

#!/usr/bin/python

# Copyright Jon Berg , turtlemeat.com

# Modified by nikomu @ code.google.com


import string,cgi,time

from os import curdir, sep

from BaseHTTPServer import BaseHTTPRequestHandler, HTTPServer



import os # os. path

CWD = os.path.abspath('.')
## print CWD

# PORT = 8080
UPLOAD_PAGE = 'upload.html' # must contain a valid link with address and port of the server


def make_index( relpath ):

abspath = os.path.abspath(relpath) # ; print abspath
flist = os.listdir( abspath ) # ; print flist

rellist = []
for fname in flist :
relname = os.path.join(relpath, fname)
rellist.append(relname)

# print rellist
inslist = []
for r in rellist :
inslist.append( '<a href="%s">%s

Unknown end tag for &lt;/a&gt;

<br>' % (r,r) )

# print inslist

page_tpl = "<html><head>

Unknown end tag for &lt;/head&gt;

<body>%s

Unknown end tag for &lt;/body&gt;



Unknown end tag for &lt;/html&gt;

"

ret = page_tpl % ( '\n'.join(inslist) , )

return ret


# -----------------------------------------------------------------------

class MyHandler(BaseHTTPRequestHandler):



def do_GET(self):

try:


if self.path == '/' :
page = make_index( '.' )
self.send_response(200)

self.send_header('Content-type',    'text/html')

self.end_headers()

self.wfile.write(page)
return


if self.path.endswith(".html"):

## print curdir + sep + self.path
f = open(curdir + sep + self.path)

#note that this potentially makes every file on your computer readable by the internet



self.send_response(200)

self.send_header('Content-type',    'text/html')

self.end_headers()

self.wfile.write(f.read())

f.close()

return


if self.path.endswith(".esp"):   #our dynamic content

self.send_response(200)

self.send_header('Content-type',    'text/html')

self.end_headers()

self.wfile.write("hey, today is the" + str(time.localtime()[7]))

self.wfile.write(" day in the year " + str(time.localtime()[0]))

return


else : # default: just send the file

filepath = self.path[1:] # remove leading '/'

f = open( os.path.join(CWD, filepath), 'rb' )
#note that this potentially makes every file on your computer readable by the internet



self.send_response(200)

self.send_header('Content-type',    'application/octet-stream')

self.end_headers()

self.wfile.write(f.read())

f.close()

return


return # be sure not to fall into "except:" clause ?


except IOError as e :
# debug
print e
self.send_error(404,'File Not Found: %s' % self.path)





def do_POST(self):

# global rootnode ## something remained in the orig. code
try:
ctype, pdict = cgi.parse_header(self.headers.getheader('content-type'))


if ctype == 'multipart/form-data' :

# original version :

'''
query=cgi.parse_multipart(self.rfile, pdict)

upfilecontent = query.get('upfile')

print "filecontent", upfilecontent[0]

'''

# using cgi.FieldStorage instead, see
# http://stackoverflow.com/questions/1417918/time-out-error-while-creating-cgi-fieldstorage-object
fs = cgi.FieldStorage( fp = self.rfile,
headers = self.headers, # headers_,
environ={ 'REQUEST_METHOD':'POST' } # all the rest will come from the 'headers' object,
# but as the FieldStorage object was designed for CGI, absense of 'POST' value in environ
# will prevent the object from using the 'fp' argument !
)
## print 'have fs'

else: raise Exception("Unexpected POST request")


fs_up = fs['upfile']
filename = os.path.split(fs_up.filename)[1] # strip the path, if it presents
fullname = os.path.join(CWD, filename)

# check for copies :
if os.path.exists( fullname ):
fullname_test = fullname + '.copy'
i = 0
while os.path.exists( fullname_test ):
fullname_test = "%s.copy(%d)" % (fullname, i)
i += 1
fullname = fullname_test

if not os.path.exists(fullname):
with open(fullname, 'wb') as o:
# self.copyfile(fs['upfile'].file, o)
o.write( fs_up.file.read() )


self.send_response(200)

self.end_headers()


self.wfile.write("<HTML><HEAD>

Unknown end tag for &lt;/HEAD&gt;

<BODY>POST OK.<BR><BR>");

self.wfile.write( "File uploaded under name: " + os.path.split(fullname)[1] );

self.wfile.write(  '<BR><A HREF=%s>back</A>' % ( UPLOAD_PAGE, )  )
self.wfile.write("

Unknown end tag for &lt;/BODY&gt;



Unknown end tag for &lt;/HTML&gt;

");




except Exception as e:

# pass

print e
self.send_error(404,'POST to "%s" failed: %s' % (self.path, str(e)) )



def main():


try:

server = HTTPServer(('', 8080), MyHandler)

print 'started httpserver...'

server.serve_forever()

except KeyboardInterrupt:

print '^C received, shutting down server'

server.socket.close()



if __name__ == '__main__':

main()


```

Despite the `<code>` tags, the above code could be distorted by the Google Wiki rendering engine, so here is a [link](http://python-simple-fileserver.googlecode.com/files/webserver.py) to the original file. (At the moment I do not see any point in putting it under the source control Ж:-) )

In addition, you will need the [upload.html](http://python-simple-fileserver.googlecode.com/files/upload.html) file -- put it in the same folder as the "webserver.py" file, and run it with "python webserver.py" from the same folder.
Now point your browser to "127.0.0.1:8080/" to test things (Linux does not allow an ordinary user to take ports below 1024, so I have used port 8080) ; later don't forget to edit (or re-generate, etc) the "upload.html" file edit to replace the "127.0.0.1" address for the one by which your computer is accessed directly !

# Links and Notes #

This wiki:

  * [Downloads](http://code.google.com/p/python-simple-fileserver/downloads/list)
  * [cgi.FieldStorage missing manual](FieldStorage.md) Ж:-)
  * [Exchanging files with a Symbian device](SymbianFileExchange.md)

External links/refs :

  * [Making a simple web server in Python](http://fragments.turtlemeat.com/pythonwebserver.php) -- main reference ;
  * [The hint for FieldStorage use](http://stackoverflow.com/questions/1417918/time-out-error-while-creating-cgi-fieldstorage-object) -- for those who do not carefully read the manuals Ж;-)
  * [Some hints, may be obsolete](http://twistedmatrix.com/pipermail/twisted-python/2004-May/007709.html), on using <tt>FieldStorage</tt> vs. <tt>parse_multipart</tt>
  * may be will add some more links later
