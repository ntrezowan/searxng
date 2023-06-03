**This instance (VM.Standard.A1.Flex) is in Oracle OCI

## How to recreate SearxNG instance

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
5. Update .env

    ```
    SEARXNG_HOSTNAME=331221.xyz
    LETSENCRYPT_EMAIL=
    ```

6. Update secret

    ```
    sed -i "s|ultrasecretkey|$(openssl rand -hex 32)|g" searxng/settings.yml
    ```

7. Update setting.yml

    ```
    use_default_settings: true

    general:
      debug: false
      instance_name: "Search"
      donation_url: false
      contact_url: false
      enable_metrics: false

    search:
      safe_search: 1
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
      secret_key: ""
      limiter: false
      image_proxy: true

      ui:
      static_use_hash: true
      default_locale: "en"
      query_in_title: false
      infinite_scroll: true
      center_alignment: true
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

8. Create systemd service

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
  
 9. Start SearxNG

    ```
    sudo systemctl enable $(pwd)/searxng-docker.service
    sudo systemctl start searxng-docker.service
    ```
