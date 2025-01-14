# setup wingetty on k3s server
* https://github.com/thilojaeggi/WinGetty
* https://www.wingetty.dev/

```

genpasswd ()
{ 
    local l=$1;
    [ "$l" == "" ] && l=30;
    tr -dc A-Za-z0-9_=., < /dev/urandom | head -c ${l} | xargs
}

test -d runtime && echo ERROR remove ./runtime dir first
test -d ./runtime && ERROR: remove ./runtime first, it seems that ther is already an existing configuration
test -d ./runtime && sleep 1h
mkdir ./runtime
chmod 700 ./runtime
export FQDN=wingetty.domain.tld
export NAMESPACE=wingetty
export REPO_NAME=REPO1
export MYSQL_USER=wg
export MYSQL_DATABASE=$REPO_NAME
export MYSQL_PASSWORD="$(genpasswd)"
export WINGETTY_SECRET_KEY="$(genpasswd)"

echo "
# keep this in a secure place
FQDN=$FQDN
NAMESPACE=$NAMESPACE
REPO_NAME=$REPO_NAME
MYSQL_DATABASE=$MYSQL_DATABASE
MYSQL_PASSWORD=$MYSQL_PASSWORD
WINGETTY_SECRET_KEY=$WINGETTY_SECRET_KEY
" > ./runtime/vars_backup.yml

kubectl create namespace $NAMESPACE
kubectl config set-context --current --namespace $NAMESPACE

for y in *.yml
do
  cp -av $y ./runtime/$y
  sed -i "s/__NAMESPACE__/$NAMESPACE/g"                     ./runtime/$y
  sed -i "s/__FQDN__/$FQDN/g"                               ./runtime/$y
  sed -i "s/__REPO_NAME__/$REPO_NAME/g"                     ./runtime/$y
  sed -i "s/__MYSQL_DATABASE__/$MYSQL_DATABASE/g"           ./runtime/$y
  sed -i "s/__MYSQL_USER__/$MYSQL_USER/g"                   ./runtime/$y
  sed -i "s/__MYSQL_PASSWORD__/$MYSQL_PASSWORD/g"           ./runtime/$y
  sed -i "s/__WINGETTY_SECRET_KEY__/$WINGETTY_SECRET_KEY/g" ./runtime/$y
done
```

# apply config in kubernetes
Finally, you can change into ./runtime directory and apply the configuration.
If needed, its also easy to adapt some changes

```
for y in 0*.yml
do
  echo $y
  kubectl apply -f $y
done
```
