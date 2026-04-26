# Mini rapport d'analyse statique APK

**Auteur :** CHARRAJ Mouad aka ZERO-XR7  
**Outil principal :** MobSF v4.0.6  
**Date d'analyse :** 26/04/2026

## 1. Contexte
Cette analyse statique a ete realisee sur l'APK `phone-check-and-test.apk` dans un cadre pedagogique, via l'environnement Mobexler et l'outil MobSF. L'objectif etait d'identifier les points de faiblesse visibles sans execution de l'application.

## 2. Informations generales
- **Nom de l'application :** Phone Check and Test
- **Package :** `com.inpocketsoftware.andTest`
- **Activite principale :** `com.inpocketsoftware.andTest.ScrollingActivityStart`
- **Version Android de l'application :** 14.4
- **Version code :** 144
- **Target SDK :** 31
- **Min SDK :** 17
- **Taille de l'APK :** 3.35 MB
- **Score MobSF :** 55/100
- **SHA-256 :** `5790b3127d661e2dab4ca73d691ff4093f13b9bdebabec0ef74adfb762944cf2`

## 3. Resume executif
L'application presente un niveau de risque **moyen**. Le rapport MobSF met surtout en avant trois points sensibles : l'usage d'un mode de chiffrement CBC avec padding PKCS5/PKCS7, l'ecriture et la lecture de donnees sur le stockage externe, ainsi qu'une compatibilite avec des versions Android anciennes et vulnnerables. Les permissions observees montrent aussi un acces a plusieurs ressources sensibles comme la localisation, la camera, le stockage et l'etat du telephone.

## 4. Resultats principaux

### 4.1 Chiffrement CBC avec PKCS5/PKCS7
- **Severite :** Elevee
- **Reference :** CWE-649, OWASP MASVS `MSTG-CRYPTO-3`
- **Constat :** MobSF signale l'usage du mode CBC avec padding PKCS5/PKCS7 dans `com/inpocketsoftware/andTest/ScrollingActivitySysInfoSensors.java`.
- **Risque :** Ce choix cryptographique ne garantit pas l'integrite des donnees chiffrees et peut exposer l'application a des attaques de type padding oracle selon le contexte d'utilisation.
- **Recommandation :** Remplacer cette implementation par un mode chiffre authentifie, par exemple AES-GCM, avec gestion correcte des IV et de l'integrite.

### 4.2 Lecture/ecriture sur le stockage externe
- **Severite :** Moyenne
- **Reference :** CWE-276, OWASP Top 10 Mobile M2, OWASP MASVS `MSTG-STORAGE-2`
- **Constat :** MobSF indique que l'application peut lire et ecrire sur le stockage externe. Les traces pointent notamment vers `com/inpocketsoftware/andTest/aa.java` et `com/inpocketsoftware/andTest/f.java`.
- **Risque :** Les donnees enregistrees sur un stockage externe peuvent etre lues, modifiees ou supprimees par d'autres applications disposant des droits adequats.
- **Recommandation :** Eviter le stockage externe pour les donnees sensibles et privilegier le stockage interne prive ou un mecanisme chiffre cote application.

### 4.3 Compatibilite avec des versions Android anciennes
- **Severite :** Elevee
- **Reference :** Manifest Analysis MobSF
- **Constat :** L'application peut etre installee a partir d'Android 4.2-4.2.2 via un `minSdk=17`, ce qui laisse ouverte l'execution sur des systemes ne recevant plus de correctifs suffisants.
- **Risque :** L'application peut tourner dans un environnement deja vulnerables, ce qui augmente la surface d'attaque globale meme si le code applicatif n'est pas directement fautif.
- **Recommandation :** Relever le `minSdkVersion` vers une version Android plus recente et alignee avec un niveau de correctifs acceptable.

### 4.4 Signature v1 exposee a Janus
- **Severite :** Moyenne
- **Reference :** Certificate Analysis
- **Constat :** MobSF signale une signature v1 pouvant rendre l'application vulnerable a Janus sur certains terminaux Android 5.0 a 8.0 si elle n'est signee qu'avec ce schema.
- **Risque :** Un attaquant pourrait profiter d'un ancien schema de signature pour contourner certaines verifications sur des versions Android precises.
- **Recommandation :** Signer l'application avec des schemas modernes v2/v3 et verifier la configuration de build de publication.

### 4.5 Journalisation d'informations sensibles
- **Severite :** Information / a surveiller
- **Reference :** CWE-532, OWASP MASVS `MSTG-STORAGE-3`
- **Constat :** MobSF releve des traces de journalisation dans `AudioTest.java`, `ScrollingActivityResults.java` et `a.java`.
- **Risque :** Des informations de debug ou des donnees applicatives peuvent fuiter dans les logs et etre recuperees sur un appareil compromis ou en phase de test.
- **Recommandation :** Nettoyer les logs avant diffusion et desactiver toute journalisation sensible en build de production.

## 5. Permissions et composants observes
### Permissions sensibles relevees
- `android.permission.ACCESS_FINE_LOCATION`
- `android.permission.CAMERA`
- `android.permission.READ_EXTERNAL_STORAGE`
- `android.permission.READ_PHONE_STATE`

### Permissions reseau et systeme visibles
- `android.permission.INTERNET`
- `android.permission.ACCESS_NETWORK_STATE`
- `android.permission.ACCESS_WIFI_STATE`
- `android.permission.BLUETOOTH`
- `android.permission.BLUETOOTH_ADMIN`
- `android.permission.NFC`

### Composants
- **Activities detectees :** 26
- **Services detectes :** 0
- **Receivers detectes :** 0
- **Providers detectes :** 1
- **Browsable activities :** aucune visible dans le rapport
- **Composants exportes visibles :** aucun export explicite releve sur les captures exploitees

## 6. Configuration reseau et ressources
- La section **Network Security** ne remonte pas d'alerte particuliere sur les captures disponibles.
- La section **Domain Malware Check** ne signale pas de domaine marque comme malveillant.
- Domaines visibles dans le rapport : `inpocketsoftware.com`, `www.inpocketsoftware.com`, `open.spotify.com`, `play.google.com`, `www.w3.org`.
- La section **Possible Hardcoded Secrets** affiche surtout des chaines liees a la localisation de l'application, ce qui ressemble davantage a des faux positifs qu'a de vrais secrets exploitables.

## 7. Correlation rapide avec OWASP MASVS
| Observation | Reference MASVS | Interpretation |
|---|---|---|
| Journalisation d'informations applicatives | `MSTG-STORAGE-3` | Les donnees sensibles ne devraient pas se retrouver dans les logs |
| Stockage externe accessible | `MSTG-STORAGE-2` | Le stockage des donnees doit limiter l'exposition aux autres applications |
| Chiffrement CBC avec PKCS5/PKCS7 | `MSTG-CRYPTO-3` | Les mecanismes cryptographiques doivent proteger confidentialite et integrite |

## 8. Recommandations prioritaires
1. Remplacer le chiffrement CBC/PKCS5-PKCS7 par une solution moderne de type AES-GCM.
2. Supprimer l'usage du stockage externe pour toute donnee sensible ou technique.
3. Relever le `minSdkVersion` pour eviter l'installation sur des versions Android trop anciennes.
4. Nettoyer les traces de log avant livraison et desactiver les logs sensibles en production.
5. Verifier la signature finale de l'APK et privilegier les schemas v2/v3.

## 9. Conclusion
L'analyse montre une application fonctionnelle mais encore marquee par plusieurs choix techniques perfectibles du point de vue securite. Le risque principal ne tient pas a un unique defaut critique, mais a l'accumulation de mauvaises pratiques sur le chiffrement, le stockage et la compatibilite avec des plateformes anciennes. Une correction de ces points permettrait d'ameliorer sensiblement le niveau de securite global.

## 10. Captures

### Connexion et lancement

![01-login-mobsf](https://github.com/user-attachments/assets/e4ac67b5-a5a6-47fb-9006-d0c9baa33004)
*Figure 1 - Ecran de connexion a MobSF.*

![02-hash-sha256](https://github.com/user-attachments/assets/d2e17657-6795-4eef-897d-25a7d870ae58)
*Figure 2 - Calcul de l'empreinte SHA-256 de l'APK analyse.*

![03-analyse-en-cours](https://github.com/user-attachments/assets/d7ccb8de-13c6-427f-859e-a9e66347c086)
*Figure 3 - Lancement de l'analyse statique de l'APK dans MobSF.*

### Tableau de bord et permissions

![04-tableau-de-bord](https://github.com/user-attachments/assets/88c1fa18-e863-489b-9288-e5b95a131a19)
*Figure 4 - Vue generale du rapport MobSF avec le score de securite et les informations principales de l'application.*

![05-permissions](https://github.com/user-attachments/assets/40520918-68bf-4405-ae45-2da40f740e38)
*Figure 5 - Liste des permissions demandees par l'application, dont plusieurs permissions sensibles.*

![06-android-api](https://github.com/user-attachments/assets/13c0cb89-179b-47c6-932d-337b779d020a)
*Figure 6 - APIs Android detectees lors de l'analyse statique.*

### Analyse securite

![07-browsable-network](https://github.com/user-attachments/assets/ffb52aff-79ea-4c60-8ad9-a75be3fbc969)
*Figure 7 - Sections Browsable Activities et Network Security, sans resultat critique visible sur cette capture.*

![08-certificat](https://github.com/user-attachments/assets/47794a57-4e10-4fdd-88c7-ea1813efa003)
*Figure 8 - Analyse du certificat et avertissement sur la signature v1 exposee a la vulnerabilite Janus.*

![10-code-analysis](https://github.com/user-attachments/assets/d7e0565e-acf6-4b50-9d5c-93f26016b068)
*Figure 9 - Resultats de l'analyse de code avec les alertes principales relevees par MobSF.*

![11-domain-malware-check](https://github.com/user-attachments/assets/d1625c07-8279-4b93-bc39-b724147edfdf)
*Figure 10 - Verification des domaines references par l'application.*

![12-hardcoded-secrets](https://github.com/user-attachments/assets/85a10eaa-fab1-4198-81f6-b9aad65a9369)
*Figure 11 - Chaines detectees comme secrets potentiels par MobSF, a interpreter avec prudence.*

![13-activities](https://github.com/user-attachments/assets/321313d1-a10d-41ca-8907-99f8666c3fef)
*Figure 12 - Liste des activites declarees dans l'application.*

