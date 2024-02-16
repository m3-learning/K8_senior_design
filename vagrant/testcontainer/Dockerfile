FROM ubuntu:latest
# LABEL maintainer=
# LABEL version=
# LABEL description=

RUN apt-get update
RUN apt-get upgrade -y

RUN apt-get install -y build-essential

RUN apt update && apt install  openssh-server sudo -y

RUN mkdir -p /home/root/.ssh
COPY id_rsa.pub /home/root/.ssh/authorized_keys

# Install Docker
RUN apt-get install -y ca-certificates curl gnupg lsb-release
RUN mkdir -m 0755 -p /etc/apt/keyrings
RUN curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
RUN echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

RUN apt-get update
RUN apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

RUN service ssh start
EXPOSE 22

# Install sudo, and set the users
RUN apt-get -y install sudo

CMD ["/usr/sbin/sshd","-D"]

ENTRYPOINT ["/lib/systemd/systemd"]
