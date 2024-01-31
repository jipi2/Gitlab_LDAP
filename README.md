# Gitlab + Integrare LDAP

## Instalare Gitlab pe Ubuntu 22

Primul pas este cel de descarcare a scriptului de la Gitlab pentru pregatirea instalarii. Se da comanda:
```
curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | sudo bash
```
Comanda __curl__ descarca scriptul, iar __sudo bash__ il ruleaza.

Gitlabul descarcat este cel __Community Edition__.

In continuare, trebuie data comanda pentru instalare:
```
sudo apt-get install gitlab-ce
```
In acest moment Gitlab este instalat.

## Modificare URL
Pentru a accesa Gitlabul din browser, este nevoie sa facem o modificare in fisierul: __/etc/gitlab/gitlab.rb__ si anume, trebuie sa schimbam valoarea variabilei *external_url*. In mod normal aici trebuie pus numele domeniului de unde dorim sa accesam Gitlab, dar noi dorind sa accesam local, vom pune adresa noastra IP.

Pentru a afla adresa IP putem da comanda: 
```
ip addr
```

O alternativa pentru aceasta actiune, este de a rula scriptul __replace_url_for_gitlab.py__ cu drepturi de root. Acest script, in python, va face aceasta modificare, in mod automat pentru noi. 
```
sudo python3 replace_url_for_gitlab.py
```
In cazul in care interfata masinii nu se numeste *ens33*, atunci va trebui modificata variabila globala din script __net_interface__.
## Pornire Gitlab
Este nevoie de 2 comenzi, pentru a salva modificarea facuta anterior:
```
sudo gitlab-ctl reconfigure
```
Dupa care:
```
sudo gitlab-ctl restart
```
## Modificare Parola Administrator
In continuare vom modifica parola Administratorului:
```
sudo gitlab-rake "gitlab:password:reset"
```
Ni se va cere username-ul user-ului caruia dorim sa ii aplicam modificarea, adica in cazul nostru __root__.
## Accesare Gitlab
Pentru a accesa gitlabul, trebuie doar sa scriem in browser adresa IP a masinii.
Ar trebui sa apara un form de sign in. Pentru a ne conecta ca Administrator, este nevoie de:
- username: __root__
- password: __parola configurata anterior__ .

## Integrare LDAP
Pentru a integra serviciul de LDAP, este nevoie sa accesam fisierul __/etc/gitlab/gitlab.rb__ pentru a modifica anumite campuri manual.
Pentru a accesa fisierul putem folosi nano, vim, geddit, etc. Singura cerinta este cea de a avea drepturi de root.

### Modificari Fisier:
Campurile pe care le voi enumera, vor trebui decomentate. In unele cazuri va trebui modificata si valoarea campului, unde este nevoie va fi specificat.

Campuri:
- gitlab_rails['ldap_enabled'] = true
- gitlab_rails['prevent_ldap_sign_in'] = false
- gitlab_rails['ldap_servers'] = YAML.load <<-'EOS'
- main: # 'main' is the GitLab 'provider ID' of this LDAP server
- label: 'LDAP'
- host: '_your_ldap_server' 
    - In loc de '_your_ldap_server', trebuie precizata adresa IP sau numele domeniului a serverului LDAP
- port: 389
    - Acest port trebuie modificat doar in cazul in care LDAP ruleaza pe un port diferit, sau este configurat sa suporte trafic criptat (TLS)
- uid: 'sAMAccountName'
    - Ce vor folosi utilizatorii pentru 'username' cand se autentifica
    - Nu trebuie modificat, dar daca se doreste ar trebui sa fie 'uid' sau 'userPrincipalName'
- bind_dn: '_the_full_dn_of_the_user_you_will_bind_with'
    - Aici trebuie modificat '_the_full_dn_of_the_user_you_will_bind_with' cu Distinguished Name-ul utilizatorului folosit pentru conectare
    - Exemplu: CN=Gitlab,OU=Users,DC=domain,DC=com
    - Utilizatorul ar trebui sa aiba drepturi de Read si Bind
- password: '_the_password_of_the_bind_user'
    - Trebuie inlocuit '_the_password_of_the_bind_user' cu parola utilizatorului.
- encryption: 'plain' # "start_tls" or 'simple_tls' 
- verify_certificates: false
    - Doar daca rulam local toata infrastructura si nu avem nevoie de securitate
- active_directory: true
    - Daca LDAP-ul este pe un AD
    - Daca nu, atunci trebuie setat pe false
- base: ''
    - Aici trebuie specificat unde se vor gasi utilizatorii
    - exemplu: 'ou=people,dc=gitlab,dc=example' sau 'DC=mydomain,DC=com'
- user_filter:''
    - Campul acesta filtreaza utilizatorii LDAP
    - Exemplu: ''(employeeType=developer)' sau '(memberOf=CN=grp.gitlab,OU=ou.gitlab,DC=ProiectDomain,DC=local)'
    - Daca nu dorim filtrare, nu este nevoie sa il decomentam.
- EOS

Exista multe optiuni care pot fi decomentate. Puteti explora.

Trebuie rulate din nou comenzile:
```
sudo gitlab-ctl reconfigure
```
```
sudo gitlab-ctl restart
```

## Verificare Conexiune LDAP
Pentru a verifica conexiunea LDAP putem rula comanda:
```
sudo gitlab-rake gitlab:ldap:check
```
In interfata, ar trebui sa apara si posibilitate de autentificare prin LDAP.

## Probleme:
Daca nu functioneaza conexiunea, se poate incerca urmatoare solutie:
- Accesarea fisierului: __/opt/gitlab/embedded/lib/ruby/gems/3.1.0/gems/net-ldap-0.17.1/lib/net/ldap.rb__
- Setati DefaultForceNoPage = true

Rualti din nou comenzile:
```
sudo gitlab-ctl reconfigure
```
```
sudo gitlab-ctl restart
```