
```
sudo apt-get install libapr1-dev libaprutil1-dev
wget http://mirror.cc.columbia.edu/pub/software/apache//httpd/httpd-2.4.9.tar.gz
tar -zxf httpd-2.4.9.tar.gz
cd httpd-2.4.9
wget https://thread-sanitizer.googlecode.com/svn/trunk/benchmarks/apache/run.sh
# Edit run.sh to set CLANG_BIN!
./run.sh
```

Config lines to play with in httpd.conf to change the number of threads/processes:
```
ServerLimit         1
StartServers        1
ThreadsPerChild    16
```
<a href='Hidden comment: 
run.sh doesn"t instrument libapr yet, ignore the lines below.
```
wget http://apache.mirrors.tds.net//apr/apr-1.5.0.tar.gz
```
'></a>