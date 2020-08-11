# Xilinx Docker installation guide<br /><br />
The installation steps are as below:<br />
1. Install Docker<br />
```
sudo apt-get install docker.io
```
2. Activate docker after installing finish<br />
```
sudo service docker start
```
3. Check docker version and the log should be liked below<br />
```
sudo docker version

Client:
 Version:           18.09.5
 API version:       1.39
 Go version:        go1.10.8
 Git commit:        e8ff056dbc
 Built:             Thu Apr 11 04:44:28 2019
 OS/Arch:           linux/amd64
 Experimental:      false
Server: Docker Engine - Community
 Engine:
  Version:          18.09.5
  API version:      1.39 (minimum version 1.12)
  Go version:       go1.10.8
  Git commit:       e8ff056
  Built:            Thu Apr 11 04:10:53 2019
  OS/Arch:          linux/amd64
  Experimental:     false
```



## Reference<br />
https://github.com/Xilinx/Vitis-AI/tree/master/docker<br />
https://github.com/Xilinx/Vitis-AI/tree/master/doc/install_docker<br />
