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
4. Add user to docker group, $user is your user name<br />
```
sudo usermod -aG docker $USER

#For example:
sudo usermod -aG docker hill213
```
5.Install NVIDIA Docker Runtime and some docker settings are need to configuration<br />
```
sudo apt-get install nvidia-container-runtime

##Systemd drop-in file
sudo mkdir -p /etc/systemd/system/docker.service.d
sudo gedit /etc/systemd/system/docker.service.d/override.conf
#Write the below setting to override.conf
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd --host=fd:// --add-runtime=nvidia=/usr/bin/nvidia-container-runtime
##
sudo systemctl daemon-reload
sudo systemctl restart docker

##Daemon configuration file
sudo gedit /etc/docker/daemon.json
#Write the below setting to daemon.json
{
    "runtimes": {
        "nvidia": {
            "path": "/usr/bin/nvidia-container-runtime",
            "runtimeArgs": []
        }
    }
}
##
sudo pkill -SIGHUP dockerd
```
6. Go to <Vitis-AI-download_directory>/docker folder and install Xilinx docker gpu version
```
./docker_build_gpu.sh
```

## Reference<br />
https://github.com/Xilinx/Vitis-AI/tree/master/docker<br />
https://github.com/Xilinx/Vitis-AI/tree/master/doc/install_docker<br />
