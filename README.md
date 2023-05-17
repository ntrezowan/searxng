
## How to recreate this site

1. Install Docker

    ```
    sudo apt update
    sudo apt install apt-transport-https ca-certificates curl software-properties-common
    sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
    sudo echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    sudo apt update
    sudo apt install docker-ce
    sudo usermod -aG docker ${USER}
    ```

2. Install Docker Compose

    ```
    mkdir -p ~/.docker/cli-plugins/
    curl -SL https://github.com/docker/compose/releases/download/v2.3.3/docker-compose-linux-x86_64 -o ~/.docker/cli-plugins/docker-compose
    chmod +x ~/.docker/cli-plugins/docker-compose
    ```

3. Open firwall for SSH and HTTP/HTTPS traffic

    ```
    sudo ufw enable
    sudo ufw allow 22
    sudo ufw allow 80
    sudo ufw allow 443
    sudo ufw status verbose
    ```
 
4. Clone SearxNG repo

    ```
    cd /usr/local
    git clone https://github.com/searxng/searxng-docker.git
    cd searxng-docker
    ```
5. Update .env file

    ```
    SEARXNG_HOSTNAME=331221.xyz
    LETSENCRYPT_EMAIL=
    ```

6. Update secret

    ```
    sed -i "s|ultrasecretkey|$(openssl rand -hex 32)|g" searxng/settings.yml
    ```

