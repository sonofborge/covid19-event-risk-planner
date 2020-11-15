# COVID19 risk planner R-Shiny application

## Installation

### Docker and Compose

1.  Pull down the repo (specifically, the `sonofborge` branch) and change into the project directory.

    ```sh
    git clone -b sonofborge https://github.com/sonofborge/covid19-event-risk-planner.git && cd covid19-event-risk-planner
    ```

1.  Create and modify `.env` for your needs.

    ```sh
    cp .env.example .env
    ```

1.  Build the Docker image (this will take a while, so grab some :coffee:).

    ```sh
    docker build -t covid19risk .
    ```

1.  Run Docker Compose.

    ```sh
    docker-compose up -d
    ```

#### First Time Running

If this is the first time running the application,
you will need to run several scripts inside the container.

1.  Hop inside the newly-created container...

    ```sh
    docker exec -it covid19risk bash
    ```

1.  And run the following scripts.

    ```sh
    /srv/shiny-server/makeDailyMaps.sh 0 
    /srv/shiny-server/makeDailyPlots.sh
    /srv/shiny-server/update_current.sh
    /srv/shiny-server/update_daily.sh
    ```

1.  Now,
    the application should be fully provisioned and running.

### Locally

`COVID19 risk planner R-Shiny application` is an R-Shiny application that requires R 3.4+, shinyserver, and several R packages.  Optionally, you can deploy behind a webserver (Apache or NGINX) to act as the reverse proxy and handle SSL termination.

**From Github**
```bash
git clone git@github.com:appliedbinf/covid19-event-risk-planner.git 
sudo mv covid19-event-risk-planner/COVID19-Event-Risk-Planner /srv/shinyserver/COVID19-Event-Risk-Planner
# Install R and Shiny Server with your distro package manager
# See https://rstudio.com/products/shiny/download-server/ for
# Shiny Server information

Rscript -e "install.packages(c( 'shiny', 'shinythemes', \
'ggplot2', 'ggthemes', 'ggpubr', 'ggrepel', 'dplyr', \
'lubridate', 'matlab'))"

```

### Requirements
* wget
* coreutils
* R 3.4+
* Shiny
* shinythemes
* ggplot2
* ggthemes
* ggpubr
* ggrepel
* dplyr
* lubridate
* matlab (R package)


### Crontab entries
We update the current data for the application every hour (with a random delay between 0-700 seconds) and the daily data every 4 hours.  While we could load the data live from the API, we want the application to be functional and responsive even if the API is down/slow and it ensures we're not hammering the API if we get a spike of users.  The real time daily data doesn't change often enough for this to introduce major delays in reporting.  

```
1 * * * * perl -le 'sleep rand 700' && /srv/shinyserver/COVID19-Event-Risk-Planner/update_current.sh
1 */4 * * * /srv/shinyserver/COVID19-Event-Risk-Planner/update_daily.sh

```
### Example Apache config
```
<VirtualHost *:80>

ServerName example.com
Redirect permanent / https://example.com/

</VirtualHost>

<VirtualHost *:443>
ServerName example.com
ErrorLog /var/logs/covid19risk/logs/error_log

SSLEngine on
SSLCertificateFile /etc/httpd/ssl/example.com_cert.crt
SSLCertificateKeyFile /etc/httpd/ssl/example.com.key
SSLProxyEngine On
SSLProtocol all -SSLv2 -SSLv3 -TLSv1 -TLSv1.1
SSLCipherSuite HIGH:!aNULL:!MD5:!3DES
SSLHonorCipherOrder on

ProxyPreserveHost On
ProxyPass / localhost:3432/
ProxyPassReverse / localhost:3432/

</VirtualHost>

```
