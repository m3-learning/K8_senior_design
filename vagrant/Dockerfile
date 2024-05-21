FROM ubuntu:latest

RUN apt-get update
RUN DEBIAN_FRONTEND=noninteractive apt-get upgrade -y

RUN DEBIAN_FRONTEND=noninteractive apt-get install -y build-essential ansible python3 python3-apt

RUN DEBIAN_FRONTEND=noninteractive apt update && apt install openssh-server sudo snapd -y

RUN mkdir -p /root/.ssh
COPY id_rsa.pub /root/.ssh/authorized_keys

RUN service ssh start
EXPOSE 22

# Install sudo, and set the users
RUN apt-get -y install sudo

CMD ["/usr/sbin/sshd","-D"]

ENTRYPOINT ["/lib/systemd/systemd"]
