>to add credentials in config:

  
git config --global user.email "email-id"  
git config --global user.name "name"  

>to push the files to github

  
git init  
git remote add origin [https://github.com/0x-mon3L/OBSIDIAN_VAULT](https://github.com/0x-mon3L/OBSIDIAN_VAULT "https://github.com/0x-mon3l/obsidian_vault")  
git add .  
git commit -m "Initial commit of my Obsidian Vault from Mac 2015"  
git branch -M main  
git push -u origin main

#to force the commit (this will delete everything from remote repo)  
git push -f origin main  

>to push the updates

  
git add .  
git commit -m "my update here"  
git push

>to fetch the update

  
git pull origin main