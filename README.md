**Author: https://github.com/searxng/searxng-docker**

*\**This instance (VM.Standard.A1.Flex) is in Oracle OCI and running on Ubuntu.**

## How to build SearXNG

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

3. By default, an OCI instance is deny all for all incoming traffic. So open firewall for SSH and HTTP/HTTPS traffic

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
5. Update .env. In here, you need to use an email that can be verified by Let's Encrypt to assign certs

    ```
    SEARXNG_HOSTNAME=331221.xyz
    LETSENCRYPT_EMAIL=who@331221.xyz
    ```

6. Update secret

    ```
    sed -i "s|ultrasecretkey|$(openssl rand -hex 32)|g" searxng/settings.yml
    ```

7. Update setting.yml as following

    ```
    # see https://docs.searxng.org/admin/engines/settings.html#use-default-settings
    use_default_settings: true

    general:
      debug: false
      instance_name: "Search"
      donation_url: false
      contact_url: false
      enable_metrics: false

    search:
      safe_search: 0
      autocomplete: 'google'
      default_lang: ""
      ban_time_on_fail: 5
      max_ban_time_on_fail: 120
      suspended_times:
        SearxEngineAccessDenied: 86400
        SearxEngineCaptcha: 86400
        SearxEngineTooManyRequests: 3600
        cf_SearxEngineCaptcha: 1296000
        cf_SearxEngineAccessDenied: 86400
        recaptcha_SearxEngineCaptcha: 604800
      formats:
        - html

    server:
      # base_url is defined in the SEARXNG_BASE_URL environment variable, see .env and docker-compose.yml
      secret_key: "f9653eaf68f00f165646ca4bbbc479c6a1f2c0ab9067f2eb4fd9cb650abbda37"  # change this!
      limiter: true  # can be disabled for a private instance
      image_proxy: true

    ui:
      static_use_hash: true
      default_locale: ""
      query_in_title: false
      infinite_scroll: true
      center_alignment: true
      results_on_new_tab: true
      cache_url: https://web.archive.org/web/
      default_theme: simple
      theme_args:
        simple_style: light
    redis:
      url: redis://redis:6379/0

    categories_as_tabs:
      general:
      images:
      videos:
      news:
      map:
      music:
      it:
      science:
      files:
      social media:

    engines:
      - name: bing
        disabled: false
      - name: qwant
        disabled: true
      - name: yahoo
        disabled: false 
    ```

8. Create a systemd service as following

    ```
    [Unit]
    Description=SearXNG service
    Requires=docker.service
    After=docker.service

    [Service]
    Restart=on-failure

    Environment=SEARXNG_DOCKERCOMPOSEFILE=docker-compose.yaml

    WorkingDirectory=/usr/local/searxng-docker
    ExecStart=/usr/libexec/docker/cli-plugins/docker-compose -f ${SEARXNG_DOCKERCOMPOSEFILE} up --remove-orphans
    ExecStop=/usr/libexec/docker/cli-plugins/docker-compose -f ${SEARXNG_DOCKERCOMPOSEFILE} down

    [Install]
    WantedBy=multi-user.target
    ```
  
 9. Start SearXNG

    ```
    sudo systemctl enable $(pwd)/searxng-docker.service
    sudo systemctl start searxng-docker.service
    ```

---

## How to update SearXNG
Run the following
```
docker-compose pull
docker-compose down
docker-compose up
```
