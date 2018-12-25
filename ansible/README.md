ansible-playbook cr-droplet.yml \--ask-vault-pass
ansible-playbook cr-droplet.yml \--vault-password-file pass-ansible.txt



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


