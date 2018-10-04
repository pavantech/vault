# vault
Run vault using  Docker containers:
1. Create the vault.hcl file
2. add below content in the file.

backend "consul" {
  address = "consul:8500"
  advertise_addr = "consul:8300"
  scheme = "http"
}
listener "tcp" {
  address = "0.0.0.0:8200"
  tls_disable = 1
}
disable_mlock = true
3. create busy box to store the config file
docker create -v /config --name config busybox;
copy the vault.hcl file in config folder
docker cp vault.hcl config:/config/;
4. run the consul 
docker run -d --name consul  -p 8500:8500 consul:v0.6.4  agent -dev -client=0.0.0.0
5. run the vault server like with the consul.
 docker run -d --name vault-dev  --link consul:consul   -p 8200:8200 --volumes-from config  cgswong/vault:0.5.3 server -config=/config/vault.hcl
 6. alias vault='docker exec -it vault-dev vault "$@"'
 7. export VAULT_ADDR=http://127.0.0.1:8200
 8. vault init -address=${VAULT_ADDR} > keys.txt
 9. vault unseal -address=${VAULT_ADDR} $(grep 'Key 1:' keys.txt | awk '{print $NF}')
 10. vault unseal -address=${VAULT_ADDR} $(grep 'Key 2:' keys.txt | awk '{print $NF}')
 11. vault unseal -address=${VAULT_ADDR} $(grep 'Key 3:' keys.txt | awk '{print $NF}')
 12. vault status -address=${VAULT_ADDR}
 13. export VAULT_TOKEN=$(grep 'Initial Root Token:' keys.txt | awk '{print substr($NF, 1, length($NF)-1)}')
 14. vault auth -address=${VAULT_ADDR} ${VAULT_TOKEN}
 15. vault write -address=${VAULT_ADDR} secret/api-key value=12345678
 16. vault read -address=${VAULT_ADDR}   secret/api-key
 17. vault read -address=${VAULT_ADDR}  -field=value secret/api-key
 18. curl -H "X-Vault-Token:$VAULT_TOKEN" -XGET http://docker:8200/v1/secret/api-key
 19. curl -s -H  "X-Vault-Token:$VAULT_TOKEN"   -XGET http://docker:8200/v1/secret/api-key  | jq -r .data.value
 vault list secret
