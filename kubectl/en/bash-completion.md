# Bash-completion (Ubuntu) 

## Walkthrough 

```
apt install bash-completion
source /usr/share/bash-completion/bash_completion
# is it installed properly 
type _init_completion
```

```
# activate for all users 
kubectl completion bash | sudo tee /etc/bash_completion.d/kubectl > /dev/null

# verify with new login 
# example 
su - tln1

# zum Testen
kubectl g<TAB>
```
