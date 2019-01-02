ansible-playbook cr-droplet-gitlab.yml \--ask-vault-pass
ansible-playbook cr-droplet-gitlab.yml \--vault-password-file datainfo.retry


ssh gitlab.pic16f874.pp.ua
sudo -i
Okay to Configure GitLab (Y|n): Y
https://gitlab.mydomain.ua
root/f...z


login
create user pic/m..6
add ssh key


  registry:
    enabled: true
    host: gitlab.pic16f874.pp.ua
    port: 5005
    api_url: http://localhost:5000 # internal address to the registry, will be used by GitLab to directly communicate with API
    path: /var/opt/gitlab/gitlab-rails/shared/registry
    key: /var/opt/gitlab/gitlab-rails/certificate.key
    issuer: omnibus-gitlab-issuer

mkdir auth
docker run --rm  --entrypoint htpasswd   registry:2 -Bbn "kanoadmin" "<K@nomdm" > auth/htpasswd

docker run -d \
  -p 5000:5000 \
  --restart=always \
  --name registry \
  -v `pwd`/auth:/auth \
  -e "REGISTRY_AUTH=htpasswd" \
  -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \
  -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd \
  -v `pwd`/etc/gitlab/ssl:/certs \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/gitlab.pic16f874.pp.ua.crt \
  -e REGISTRY_HTTP_TLS_KEY=/certs/gitlab.pic16f874.pp.ua.key \
  registry:2


ansible-playbook cr-droplet-gitlab.yml --ask-vault-pass

ansible-playbook cr-droplet-gitlab.yml --vault-password-file datainfo.retry
ansible-playbook cr-droplet-gitlab.yml --vault-password-file datainfo.retry

# backup system
sudo gitlab-rake gitlab:backup:create
/var/opt/gitlab/backups/1546357720_2019_01_01_11.6.1_gitlab_backup.tar
# backup config
sudo sh -c 'umask 0077; tar -cf $(date "+etc-gitlab-%s.tar") -C / etc/gitlab'
etc-gitlab-1546358092.tar
# backup secrets gitlab-secrets.json  gitlab.rb 

# restore config
sudo tar -xf etc-gitlab-1399948539.tar -C /
sudo gitlab-ctl reconfigure

###############################
cd /var/opt/gitlab/backups/
gitlab-rake gitlab:backup:create
sh -c 'umask 0077; tar -cf $(date "+%s_%Y_%m_%d_etc_gitlab_backup.tar") -C / etc/gitlab'
###############################

ssh gitlab.oleryz79.pp.ua "sudo cp -vR /var/opt/gitlab/backups /tmp"
ssh gitlab.oleryz79.pp.ua "sudo chmod -vR 777 /tmp/backups"
scp gitlab.oleryz79.pp.ua:/tmp/backups/* .
ssh gitlab.oleryz79.pp.ua "sudo rm -vfR 777 /tmp/backups"