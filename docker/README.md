# Install Docker Engine on CentOS
## _OS requirements_

Para instalar Docker Engine, necesita una versión mantenida de CentOS 7 u 8. Las versiones archivadas no son compatibles ni probadas. El repositorio `centos-extras` debe estar habilitado. Este repositorio está habilitado de forma predeterminada, pero si lo ha deshabilitado, debe volver a habilitarlo, Se recomienda el controlador de almacenamiento overlay2.


## Uninstall old versions

Las versiones anteriores de Docker se llamaban docker o docker-engine. Si están instalados, desinstálelos, junto con las dependencias asociadas.

```sh
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine    
```

## Install the dependencies

```sh
SET_HOSTNAME="Name"
#
sudo yum update -y                                      # Update
hostnamectl set-hostname $SET_HOSTNAME                  # Set hostname
systemctl disable firewalld; systemctl stop firewalld   # Disable firewall
sed -i --follow-symlinks 's/^SELINUX=enforcing/SELINUX=disabled/' /etc/sysconfig/selinux    # Disable selinux
echo "$(ip addr | grep -Po '(?!(inet 127.\d.\d.1))(inet \K(\d{1,3}\.){3}\d{1,3})') $(hostname)" >> /etc/hosts   # Add hostname
curl -fsSL https://get.docker.com -o get-docker.sh      # Download docher script
sh get-docker.sh                                        # Install docker
systemctl start docker; sudo systemctl enable docker    # Start and enable docker
reboot       
```

> Note: `https://docs.docker.com/engine/install/centos/` este es el archivo de referencia.