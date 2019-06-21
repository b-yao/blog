# Docker Behind Proxy
Configuring docker proxy needs understanding of systemd, [this page in digitalocean][1] is a very nice one to start from. In docker official docs, the different situations of envolving [docker daemon][2] and/or [docker containers][3] are not well separated. Here we make that straight.

## Proxy for docker daemon
The command that involves docker daemon itself is the `docker pull`. This works with Docker version 18.09.6 in Ubuntu18.04, which uses systemd to manage docker daemon:
- Create a systemd drop-in directory for the Docker service, which take precedence before the `/lib/systemd/system/docker.service`
  ```
  mkdir /etc/systemd/system/docker.service.d
  tee /etc/systemd/system/docker.service.d/proxy.conf << EOF
  [Service]
  Environment="HTTP_PROXY=http://10.77.77.77:8080"
  Environment="HTTPS_PROXY=http://10.77.77.77:8080"
  Environment="NO_PROXY=localhost,127.0.0.1,192.168.*.*,172.16.*.*,172.31.*.*,10.*.*.*"
  EOF
  systemctl daemon-reload
  systemctl restart docker
  ```
- Veryfiy it by running `docker info | grep -i proxy`

## Proxy for running docker containers
- Make docker containers uses proxy, this is for `docker build`, `docker run` etc.
  ```
  mkdir ~/.docker/
  sudo tee ~/.docker/config.json << EOF
  {
    "proxies":
    {
      "default":
      {
        "httpProxy": "http://10.77.77.77:8080",
        "httpsProxy": "http://10.77.77.77:8080",
        "noProxy": "localhost,127.0.0.1,192.168.*.*,172.16.*.*,172.31.*.*,10.*.*.*"
      }
    }
  }
  EOF
  sudo systemctl restart docker
  ```
[1]: https://www.digitalocean.com/community/tutorials/understanding-systemd-units-and-unit-files
[2]: https://docs.docker.com/config/daemon/systemd/
[3]: https://docs.docker.com/network/proxy/

