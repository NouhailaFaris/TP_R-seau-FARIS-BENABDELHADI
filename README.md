# TP_Bus et réseaux industriels

# Réaliser par :

- Nouhaila FARIS
- Mohamed BENABDELHADI

## Objectif de TP:

 mettre en place l'ensemble des composants suivant:
 
 ![image](https://github.com/user-attachments/assets/7d0f6780-9d64-4791-b3f1-35c563478f4d)



 ## TP 1:Bus I2C
 
**mettre en place la communication entre le microcontrôleur et les capteurs (température, pression, accéléromètre...) via  le bus I²C.**

### Capteur BMP280 :

1. les adresses I²C possibles pour ce composant: 0x76 : lorsque SDO est connecté à la masse (GND).
0x77 : lorsque SDO est connecté à la tension d'alimentation (VDDIO).

2. le registre et la valeur permettant d'identifier ce composant: l'adresse 0xD0 , la valeur est 0x58.

3. le registre et la valeur permettant de placer le composant en mode normal:Le registre ctrl_meas à l'adresse 0xF4. Les bits doivent être à (11) ==>Mode normale.

4. les registres contenant l'étalonnage du composant:Les données d'étalonnage se trouvent dans les registres calib00 à calib25, de l'adresse 0x88 à 0xA1.

5. les registres contenant la température (ainsi que le format):
- temp_msb à l'adresse 0xFA (bits 15-8)
- temp_lsb à l'adresse 0xFB (bits 7-0)
- temp_xlsb à l'adresse 0xFC (bits 7-4)

6. les registres contenant la pression (ainsi que le format):
- press_msb à l'adresse 0xF7 (bits 15-8)
- press_lsb à l'adresse 0xF8 (bits 7-0)
- press_xlsb à l'adresse 0xF9 (bits 7-4)

7. les fonctions permettant le calcul de la température et de la pression compensées, en format entier 32 bits:

![image](https://github.com/user-attachments/assets/1762f883-cabb-47ad-8956-99a91d7de897)

### Test de la chaîne de compilation et communication UART sur USB

 ![image](https://github.com/user-attachments/assets/2d6e6b36-20b6-44f5-a11b-88a4bfdd816c)


### Communication I²C
#### Identification du BMP280


```c
// Définition de l'adresse I2C du BMP280, en décalant de 1 bit à gauche pour correspondre au format standard I2C (écriture sur 7 bits).
#define BMP280_I2C_ADDRESS 0x77 << 1 

// Définition de l'adresse du registre pour identifier le composant BMP280.
#define BMP280_ID_REG 0xD0 

// Fonction principale du programme.
int main(void) 
{ 
    // Affichage d'un message de démarrage.
    printf("=======demarage Systeme=======\r\n"); 

    // Initialisation de la variable pour contenir l'adresse du registre (identification).
    uint8_t reg = BMP280_ID_REG; 

    // Variable pour stocker l'ID du composant BMP280.
    uint8_t bmp280_id = 0; 

    // Envoi de l'adresse du registre BMP280_ID_REG (0xD0) via l'I2C pour interroger l'ID du composant.
    HAL_I2C_Master_Transmit(&hi2c1, BMP280_I2C_ADDRESS, &reg, 1, HAL_MAX_DELAY); //envoyer l'adresse du registre ID

    // Réception de 1 octet contenant l'ID du BMP280 depuis le composant via I2C.
    HAL_I2C_Master_Receive(&hi2c1, BMP280_I2C_ADDRESS, &bmp280_id, 1, HAL_MAX_DELAY); //recevoir 1 octet correspondant au contenu du registre

    // Affichage de l'ID reçu en hexadécimal.
    printf("Id: 0x%x...", buf[0]);

    // Vérification si l'ID reçu correspond à la valeur attendue pour le BMP280 (doit être 0x58).
    if (buf[0] == BMP280_ID_VAL) {
        // Si l'ID est correct, afficher "Ok" et terminer le programme avec succès.
        printf("Ok\r\n");
        return 0;
    } else {
        // Si l'ID est incorrect, afficher "not Ok!" et terminer le programme avec une erreur.
        printf("not Ok!\r\n");
        return 1;
    }
}

```
**le contenu du registre correspond bien à la datasheet:**

![image](https://github.com/user-attachments/assets/8f9a9f1e-26c8-4432-89ab-7fab195d8874)



### Configuration du BMP280

```c

// Définition de l'adresse du registre de contrôle pour les mesures.
#define BMP280_CTRL_MES_REG 0xF4 

int main(void) 
{ 

    // Affichage d'un message de démarrage.
    printf("======= Démarrage Système =======\r\n"); 

    // Variable pour contenir l'adresse du registre de contrôle.
    uint8_t reg1 = BMP280_CTRL_MES_REG; 

    // Buffer pour l'envoi des données via I2C.
    uint8_t buf[2]; 

    // Préparation des données à écrire :
    // - buf[0] contiendra l'adresse du registre de contrôle.
    // - buf[1] contiendra la valeur à écrire dans ce registre.
    buf[0] = reg1;  // Adresse du registre
    buf[1] = 0x57;  // Valeur à écrire (configurations d'oversampling et mode normal).

    // Envoi de l'adresse et de la valeur au BMP280 pour configurer le registre 0xF4.
    HAL_I2C_Master_Transmit(&hi2c1, BMP280_I2C_ADDRESS, buf, 2, HAL_MAX_DELAY); 

    // Réenvoi de l'adresse du registre pour lire sa valeur actuelle.
    HAL_I2C_Master_Transmit(&hi2c1, BMP280_I2C_ADDRESS, &reg1, 1, HAL_MAX_DELAY); 

    // Variable pour stocker la valeur lue du registre.
    uint8_t bmp280_ctrl; 

    // Lecture de la valeur actuelle du registre 0xF4.
    HAL_I2C_Master_Receive(&hi2c1, BMP280_I2C_ADDRESS, &bmp280_ctrl, 1, HAL_MAX_DELAY); 

    // Affichage de la valeur lue en hexadécimal.
    printf("Valeur du registre de contrôle (0xF4) : 0x%02X\r\n", bmp280_ctrl); 
}


```
**Vérification du configuration**

 ![image](https://github.com/user-attachments/assets/f8de9a36-e626-463b-8ec6-9aaa9afbc1b2)"

### Récupération de l'étalonnage, de la température et de la pression

 ![image](https://github.com/user-attachments/assets/136f1959-bfd9-4d44-8567-f35044e979a9)

 ### Calcul des températures et des pression compensées et Interfaçage de l'accléromètre:
 
 **D'aprés  la datasheet du STM32 on a :**
 
![WhatsApp Image 2024-11-22 at 11 54 22](https://github.com/user-attachments/assets/90361d23-e054-48c7-8fe7-8b8109d131e3)

**Résultat Final:**

![image](https://github.com/user-attachments/assets/bf406e71-3157-4ac5-8568-c7c07a1db3f0)

Nous avons réussi à afficher les valeurs compensées de la température, de la pression et de l'accéléromètre

 ## TP 2:
 
 ### Objectif: Permettre l'interrogation du STM32 via un Raspberry Pi Zero Wifi:
 
 ![image](https://github.com/user-attachments/assets/1a1c718c-3c32-4f5a-9683-fcb8866964af)

**Préparation du Raspberry**

**L'outil Rpi_Imager a été utilisé pour flasher l'image sur la carte SD.**

 ![image](https://github.com/user-attachments/assets/2697a3e5-2ef1-443b-a6ca-087e84e0d57e)
 
 ![image](https://github.com/user-attachments/assets/94f776dd-35a3-4498-9f7a-2e10c8755ed6)

 
**Premier démarrage**

**Connexion SSH :**

**L’adresse IP du Raspberry a été attribuée dynamiquement par le routeur (DHCP).**

![image](https://github.com/user-attachments/assets/06dd0bb4-5c9b-4916-a870-c02d4f86b695)

**Branchements:**
Les broches TX et RX du port UART du Raspberry Pi (GPIO 14 et GPIO 15) ont été reliées entre elles pour simuler une communication en boucle

### Configuration et test avec minicom:

**installation minicom**

```
sudo apt update
sudo apt install minicom
```
**configuration minicom**

![image](https://github.com/user-attachments/assets/7440a819-35a6-43c9-8d32-0b3d0156ebb5)

**Test minicom**

![image](https://github.com/user-attachments/assets/f2a962d3-000d-4299-931e-9e45f2eff4c5)

### Implémentation du protocole sur la STM32 :

Le code suivant a été écrit pour gérer les commandes reçues via UART et répondre conformément au protocole défini :

```
void handleCommand(char *command, BMP280_S32_t *comp_temp, BMP280_U32_t *comp_press) {
    if (strncmp(command, "GET_T", 5) == 0) {
        // Calcul de la température compensée
        *comp_temp = bmp280_compensate_T_int32(raw_temp);
        float temperature = *comp_temp / 100.0; // Conversion en °C
        snprintf(txBuffer, sizeof(txBuffer), "T=%.2f°C\r\n", temperature);
        HAL_UART_Transmit(&huart4, (uint8_t *)txBuffer, strlen(txBuffer), HAL_MAX_DELAY);
    } else if (strncmp(command, "GET_P", 5) == 0) {
        // Calcul de la pression compensée
        *comp_press = bmp280_compensate_P_int64(raw_press);
        snprintf(txBuffer, sizeof(txBuffer), "P=%ldPa\r\n", *comp_press);
        HAL_UART_Transmit(&huart4, (uint8_t *)txBuffer, strlen(txBuffer), HAL_MAX_DELAY);
    } else {
        snprintf(txBuffer, sizeof(txBuffer), "Unknown Command\r\n");
        HAL_UART_Transmit(&huart4, (uint8_t *)txBuffer, strlen(txBuffer), HAL_MAX_DELAY);
    }
```
**Test depuis le Raspberry Pi**

![image](https://github.com/user-attachments/assets/d68976d3-86b3-4700-bb16-c59c31e758a4)


### Commande depuis Python:

![image](https://github.com/user-attachments/assets/e9673bb0-6ff2-4119-9b4b-29b261d2049a)

**Script Python : Communication STM32**

```le script est dans le fichier  stm32_communication.py```

### Exécution du script :
```
python3 stm32_communication.py
```
### Résultat:

![image](https://github.com/user-attachments/assets/22a18b4a-ed37-4c0c-b799-c39519beeb05)



 ## TP 3: Développement d'une interface REST sur le Raspberry
 
### Installation :

1. Création d'un utilisateur avec droits d'administration et accès au port série
   
```c
sudo adduser nouhfari 
sudo usermod -aG sudo nouhfari  
sudo usermod -aG dialout nouhfari 
```

2. Délogguez vous, puis relogguez vous en tant que nouhfari
   
```c
logout
ssh nouhfari@192.168.88.228
```

3. Création du fichier requirement.txt

![image](https://github.com/user-attachments/assets/0e661ee0-9e57-47f6-a517-9a58d3ce1fb1)


4. Premier fichier Web:
   
**Création d’un fichier hello.py dans le répertoire ~/server**
```
nano hello.py

```
**Contenu de hello.py**
```
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello, World!\n'

```
### Lancement du serveur:

![image](https://github.com/user-attachments/assets/e88737cb-7fa9-4cff-96d1-d3b345701e84)

### Test du serveur: 
![image](https://github.com/user-attachments/assets/608c2056-3cbd-4b61-8d35-c533d10f2a03)


1. Première route:

**Rôle du décorateur @app.route**

Le décorateur @app.route est utilisé dans Flask pour associer une URL à une fonction. Chaque fois qu'un utilisateur accède à une URL correspondant à celle spécifiée par le décorateur, la fonction Python associée est exécutée.

**Rôle du fragment <int:index>**

Le fragment <int:index> dans l'URL permet à Flask de capturer un paramètre de type entier dans l'URL et de le passer à la fonction associée.

2. Test avec json.dumps()

```py
import json
from flask import Flask, abort

app = Flask(__name__)

welcome = "Welcome to 3ESE API!"

@app.route('/api/welcome/<int:index>')
def api_welcome_index(index):
    try:
        return json.dumps({"index": index, "val": welcome[index]})
    except IndexError:
        abort(404)
```

![image](https://github.com/user-attachments/assets/f4b414c0-6e15-47b0-83ca-4202d69a1bc6)

### Résultat:
![image](https://github.com/user-attachments/assets/ad501736-66cf-45b2-9a8f-d851b8dcc8dd)

On determinant l'index on peut visualiser le caractére qui correspondant dans le tableau welcome  

3. 1re solution:

**Definition JSON:**

En Python, json (JavaScript Object Notation) est un module standard utilisé pour encoder et décoder des données au format JSON, un format de texte léger pour l'échange de données. JSON est couramment utilisé pour transmettre des données entre un serveur et un client (par exemple, dans des APIs web) grâce à sa simplicité et à sa compatibilité avec de nombreux langages de programmation.

```py
from flask import Flask
import json
app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello, World!\n'

welcome = "Welcome to 3ESE API!\n"

@app.route('/api/welcome/')
def api_welcome():
    return welcome

@app.route('/api/welcome/<int:index>')
def api_welcome_index(index):
    return json.dumps({"index": index, "val": welcome[index]})

```

![image](https://github.com/user-attachments/assets/e78eabd7-24c0-4d10-9f62-f0874ecf9f83)

On peut bien remarquer que le type n'est pas du JSON c'est du html

![image](https://github.com/user-attachments/assets/e0ae9f05-016d-420f-a792-6975e0b60355)

Ici on a du Json mais c'est nous qui ont forcer ce type avec cette solution

4. 2éme solution

**teste jsonify** 

![image](https://github.com/user-attachments/assets/807e1bbe-2fc0-4a34-a5d6-55079b35b375)

![image](https://github.com/user-attachments/assets/3f0a2248-1beb-4af9-9b46-cc1bd0b2e30d)

cette solution est meilleure puiqu'on Jsonify le python donc on retrouve que notre python est convertit on json 

5. Erreur 404

### Ajout de la gestion des erreurs 404 dans hello.py:

```py
from flask import Flask, jsonify, abort

app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello, World!\n'

welcome = "Welcome to 3ESE API!\n"

@app.route('/api/welcome/')
def api_welcome():
    return welcome

@app.route('/api/welcome/<int:index>')
def api_welcome_index(index):
    if 0 <= index < len(welcome):
        return jsonify({"index": index, "val": welcome[index]})
    else:
        abort(404)

@app.errorhandler(404)
def page_not_found(error):
    return jsonify({"error": "Page not found"}), 404
```
### Test de la gestion des erreurs :

![image](https://github.com/user-attachments/assets/7e51c079-0691-45fa-8c86-e70f57b1fb7e)


### Méthodes POST, PUT, DELETE


#### API CRUD

 **Le script est dans le fichier Test_API.py**
 
![image](https://github.com/user-attachments/assets/aeda9a38-1e90-4837-82d7-43647357b26d)

![image](https://github.com/user-attachments/assets/5c4a6d79-dd20-4f58-af0f-24c28ba5f4f0)



# Tp4_Bus CAN

##  Objectif: Développement d'une API Rest et mise en place d'un périphérique sur bus CAN

### Configuration du bus CAN avec CubeMX:

![image](https://github.com/user-attachments/assets/77538c80-8f19-4b30-8ace-854c0075aa64)
![image](https://github.com/user-attachments/assets/2e20b4c8-96e1-456b-a587-8ff39b89f5c7)

### Pilotage du moteur:
![image](https://github.com/user-attachments/assets/3296caf0-9704-4a29-a0b3-8d420b9e9e24)

##  Interfaçage avec le capteur:

### Test et validation :

![image](https://github.com/user-attachments/assets/fb93f9e2-8de0-4058-938c-9c30ec7eeec7)

le moteur tourne par rapport à la variation de la température.

# TP 5
# Objectif: Faire marcher ensemble les TP 1, 2, 3 et 4

![WhatsApp Image 2024-11-22 at 18 18 49 (2)](https://github.com/user-attachments/assets/ccc37bad-d083-4b55-9e9b-e59c94e8d184)

### Mesures de température et de pression sur I²C par le STM32

 ![image](https://github.com/user-attachments/assets/136f1959-bfd9-4d44-8567-f35044e979a9)

### Communication série entre la STM32 et le Raspberry PI zero: implémentation du protocole proposé au TP2.4

![image](https://github.com/user-attachments/assets/ab208d89-f4a8-4515-aec9-2264b04cd4ed)

### API REST sur le Raspberry

Code Python:(Flask)
```
Le code est dans Test.py
```
![image](https://github.com/user-attachments/assets/dd71d9f3-59bf-4772-bbda-baa05a9ed04b)

### Test sur Web

![image](https://github.com/user-attachments/assets/5373c01d-8c5f-4577-be8a-75c4b8e56362)



















  
