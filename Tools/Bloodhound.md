# 1️⃣ Completely Remove BloodHound

## Stop services first

sudo neo4j stop

---

## Remove BloodHound + Neo4j packages

sudo apt purge bloodhound neo4j bloodhound.py -y  
sudo apt autoremove -y

---

## Clean leftover directories (IMPORTANT)

sudo rm -rf /usr/share/bloodhound  
sudo rm -rf /etc/bloodhound  
sudo rm -rf /var/lib/neo4j  
sudo rm -rf /etc/neo4j  
sudo rm -rf /usr/share/neo4j

---

## Clean cache (optional but recommended)

sudo apt clean

---

# 📥 2️⃣ Fresh Install (Recommended Method)

## Update system

sudo apt update

---

## Install Neo4j first

sudo apt install neo4j -y

---

## Install BloodHound

sudo apt install bloodhound -y

---

## Install collector (VERY IMPORTANT)

sudo apt install bloodhound.py -y


---

# ⚙️ 3️⃣ Configure Neo4j (CRITICAL STEP)

## Start Neo4j

sudo neo4j start

---

## Open in browser:

http://localhost:7474

---

## Default login:

neo4j / neo4j

👉 It will FORCE password change  
Set something like:

neo4j / Password123!

---

# 🚀 4️⃣ Start BloodHound GUI

bloodhound

👉 It will open:

http://127.0.0.1:8080

---

## Login using:

neo4j / <your-new-password>



