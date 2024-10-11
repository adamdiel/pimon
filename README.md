# PiMon
Remote speed test monitoring using cloudflare tunnels and a pi


Pre-Steps:

- Installed `ubuntu` on a Rasbperry Pi 4
- Made sure WiFi and ethernet port were setup so either one could work on reboot/pulling DHCP
- Followed [these steps](https://medium.com/@mohsentaleb/how-to-access-your-raspberry-pi-via-ssh-orvnc-from-anywhere-in-the-world-using-cloudflares-zero-9dcd2e75a9d7) to setup a Cloudflare tunnel for both HTTPS and SSH traffic into the pi. It gets installed as a service and should start on boot
  - High level you are starting the `cloudflared` service to authorize and connect to the cloudflare tunnel. Then on your client box, you'll need to connect SSH via the `cloudflared` service as well. You can set up your ssh config to do this for you as well
  - This is an example config file on the Pi looks like
    ```
    adam@crashpad:~$ cat /etc/cloudflared/config.yml
    tunnel: abc456gh-ijkl-4n0p-br8t-u4w00y5z0280
    credentials-file: /home/adam/.cloudflared/cert.pem

    ingress:
    - hostname: ssh-pi.adamdiel.com
      service: ssh://localhost:22
    - hostname: grafana-pi.adamdiel.com
      service: http://localhost:3000
    - service: http_status:404
    ```

    *Note* - I learned that you can't do `ssh.rpi.adamdiel.com` at least for ssh as cloudflare's edge certs only cover 1 level of sub domains I believe


Docker Steps:

*NOTE* - Depending on your Docker install/config, you might need to run all of these pre-fixed with `sudo`

- Modify `grafana/config/grafana.ini` with updated credentials you'll want to use to login
- Modify `ping_exporter/config/config.yml` with any additional ping targets. You can do IP or DNS. *Note* Packet loss graphs get funky when IPs change for big targets like google/cloudflare.
- Modify the `GF_SECURITY_ADMIN_PASSWORD` to a secret of your choice in the `docker-compose.yml` file.
- `docker volume create --name=prometheus-volume`
- `docker-compose up -d`
- Login to https://YOURHTTPSTUNNELDOMAIN with the creds you set in `grafana.ini`
- Go to `connections` -> `Data Sources` and add `Prometheus`. The Docker container name is the hostname by default. Verifying with a `sudo docker ps` I was able to tell my URL should be `http://pimon_prometheus_1:9090` as `pimon_prometheus_1` was my container name. Save and test should work
- You can import the example dashboard in the `grafana/example-dashboard` directory of this repo. Go to `Dashboards` -> `New` -> `Import` and copy paste the `json` into the text box and click `Load`


![image](https://github.com/user-attachments/assets/81e2282e-dec8-44fc-a45f-aa406eaeb367)





