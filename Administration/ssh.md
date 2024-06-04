# generate a new SSH key-pair using Ed25519 algorithm
ssh-keygen -t ed25519 -C "[EMAIL]"

# start the ssh-agent in the background
eval "$(ssh-agent -s)"

# add SSH private key to the ssh-agent
ssh-add ~/.ssh/id_ed25519

# test SSH connection to GitHub
ssh -T git@github.com

# connect to a server using specified private key
ssh -i /path/to/private/key username@host

# configure git to use SSH to sign commits and tags
git config --global gpg.format ssh

# set SSH signing key with for Git
git config --global user.signingkey /PATH/TO/.SSH/KEY.PUB

# sign all commits by default in any local repository
git config --global commit.gpgsign true

# create a signed commit
git commit -S -m "YOUR_COMMIT_MESSAGE"

# sets the name you want attached to your commit transactions
git config --global user.name "[USERNAME]"

# sets the email you want attached to your commit transactions
git config --global user.email "[EMAIL]"

# enables helpful colorization of command line output
git config --global color.ui auto
