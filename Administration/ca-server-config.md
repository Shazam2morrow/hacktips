CONFIGURATION OF CA SERVER

# install easy-rsa
sudo apt install easy-rsa

mkdir ~/easy-rsa

ln -s /usr/share/easy-rsa/* ~/easy-rsa/

chmod 700 /home/unclesam/easy-rsa

cd ~/easy-rsa/

./easyrsa init-pki
"init-pki complete; you may now create a CA or requests."
"Your newly created PKI dir is: /home/unclesam/easy-rsa/pki"

vim vars

set_var EASYRSA_REQ_COUNTRY    "UK"
set_var EASYRSA_REQ_PROVINCE   "Reynoldsstad"
set_var EASYRSA_REQ_CITY       "West Lola"
set_var EASYRSA_REQ_ORG        "Martin & Co"
set_var EASYRSA_REQ_EMAIL      "lwright@martin.co.uk"
set_var EASYRSA_REQ_OU         "Private"
set_var EASYRSA_ALGO           "ec"
set_var EASYRSA_DIGEST         "sha512"

./easyrsa build-ca

Enter New CA Key Passphrase: 
Re-Enter New CA Key Passphrase: 
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Common Name (eg: your user, host, or server name) [Easy-RSA CA]:

CA creation complete and you may now import and sign cert requests.
Your new CA certificate file for publishing is at:
/home/unclesam/easy-rsa/pki/ca.crt