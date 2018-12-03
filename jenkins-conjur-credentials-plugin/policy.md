_This section refers to *Enrolling an Application* at https://www.conjur.org/tutorials/policy/applications.html_

### Login to Conjur Client
`docker-compose exec client bash`{{execute}}

### Login to Conjur
```
conjur init -u conjur -a quick-start
export admin_api_key="$(cat admin_key|awk '/API key for admin/ {print $NF}'|tr '  \n\r' ' '|awk '{$1=$1};1')"
docker-compose exec client conjur init -u conjur -a quick-start
docker-compose exec client conjur authn login -u admin -p $admin_api_key
```{{execute}}

### Create policy files
```
cat >> conjur.yml << EOF
- !policy
  id: db

- !policy
  id: frontend
EOF
cat >> db.yml << EOF
- &variables
  - !variable password

- !group secrets-users

- !permit
  resource: *variables
  privileges: [ read, execute ]
  roles: !group secrets-users

# Entitlements
- !grant
  role: !group secrets-users
  member: !layer /frontend
EOF
cat >> frontend.yml << EOF
- !layer

- !host frontend-01

- !grant
  role: !layer
  member: !host frontend-01
EOF
  
```{{execute}}

### Load policies to Conjur

```
conjur policy load --replace root conjur.yml
conjur policy load frontend frontend.yml
conjur policy load db db.yml
```{{execute}}

**Copy the api_key generated!**

### Create secret for database
```
password=$(openssl rand -hex 12)
conjur variable values add db/password $password
conjur variable value db/password
```{{execute}}