# docker_download

Docker image downloader

When you call `docker pull image:tag` it will establish a connection to the docker image registry, get image layers and load them to docker.

But in some situations it's not as simple as above.
For example, the `docker.io` website sanctions or anything like that. So we need a good traditional way.
Download image layers with separate tools and feed to docker.

By using this script, `docker pull` turns to `dn_docker dir_name image_name:tag`

---

**Hint**
- This is not a miracle. I just used [Mobi's docker script](https://github.com/moby/moby/blob/master/contrib/download-frozen-image-v2.sh) with some slight improvements.
- This version only works for `docker.io` registry. I will add some other registry soon.

## Usage
To download image from a registry go to https://hub.docker.com/ and sereach for it.

For example here is hello world page:

https://hub.docker.com/_/hello-world

For default you use probably run this command:
```
docker pull hello-world:nanoserver-ltsc2025
```

But to download which this script run:
```
 ./dn_docker helloworld_dir hello-world:nanoserver-ltsc2025
```


---


Most important ones are:
- Use `aria2` downloader instead of `curl`.
- Move download command to bash function.
- Add bash commands to `download`, `archive`, `compress` and `load` docker image.
- Currently tested only on Debian 12, 13, and Ubuntu 24.04.

---

## Usage 

### Prerequisite
```
$ sudo apt install curl aria2c jq tar gzip
```

Download `dn_docker` script and make it executable
```
git clone git@github.com:esmaeelE/docker_download.git
cd docker_download
chmod +x dn_docker
```

Also you can place `dn_docker` to `$PATH`

```
sudo mv dn_docker /usr/local/bin/
```

After that, you can directly call `dn_docker` and remove the leading `./` from the commands below.

### Run
```
./dn_docker dir_name image_name:tag
```

## Use in restricted countires 

If you are unable to access the Docker registry at docker.io, some possible solutions include using a proxy or a VPN. However, addressing internet censorship is beyond the scope of this repository.


### Docker error while run docker pull

```
Error response from daemon: pull access denied for postgres, repository does not exist or may require 'docker login': denied: <html><body><h1>403 Forbidden</h1>
Since Docker is a US company, we must comply with US export control regulations. In an effort to comply with these, we now block all IP addresses that are located in Cuba, Iran, North Korea, Republic of Crimea, Sudan, and Syria. If you are not in one of these cities, countries, or regions and are blocked, please reach out to https://hub.docker.com/support/contact/
</body></html>
```


Export proxy in current shell
```
export {http,https,ftp}_proxy="10.10.34.42:8080"
./dn_docker dir_name image_name:tag
```

Utilize `proxychains` to route connections through proxy servers prior to invoking `dn_docker`.
```
proxychanis4 ./dn_docker dir_name image_name:tag
```

Or, if you are connected to a VPN server, all traffic will be routed through the established connection.


## Links

- [Baeldung post](https://www.baeldung.com/ops/docker-download-image-no-client)

