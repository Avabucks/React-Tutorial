# React Tutorial

## Installazione

Per avviare il progetto in locale, segui questi passaggi:

### Prerequisiti
Assicurati di avere Node.js e npm installati. Puoi verificare la loro versione con i seguenti comandi:

```bash
node -v
npm -v
```

### Installa Expo CLI globalmente
Usa il comando seguente per installare Expo CLI a livello globale:

```bash
npm install -g expo-cli
```

## Inizializzazione

### Creazione di un'App React (per Web)
1. Crea una nuova app React

```bash
npx create-react-app nome-progetto
```

2. Entra nella cartella del progetto
```bash
cd nome-progetto
```

3. Apri il progetto in VS Code
```bash
code .
```

4. Avvia il server di sviluppo
```bash
npm start
```

### Creazione di un'App Expo (per Android/iOS)
1. Inizializza un progetto Expo

```bash
expo init nome-progetto
```

2. Entra nella cartella del progetto
```bash
cd nome-progetto
```

3. Apri il progetto in VS Code
```bash
code .
```

4. Avvia il server di sviluppo Expo
```bash
npx expo start
```

5. Avvia il progetto in modalità Web
```bash
npx expo start --web
```

## Build APK/AAB per Android
1. Costruisci il pacchetto per Android (AAB)

```bash
eas build --platform android
```

2. Costruisci un pacchetto per Android con profilo di anteprima (APK)
```bash
eas build --platform android --profile preview
```

## Convertire un file AAB in APK
Assicurati di avere bundletool.jar scaricato e usa il comando seguente
```bash
java -jar bundletool.jar build-apks --bundle=app.aab --output=app.apks --mode=universal
```

## REST API JSON
Codice PHP per creazione JSON:

```php
<?php
header("Access-Control-Allow-Origin: *");
header("Access-Control-Allow-Methods: GET, POST, OPTIONS");
header("Access-Control-Allow-Headers: Content-Type, Authorization");
header("Content-Type: application/json");

// Un semplice array di dati
$data = [
    'messaggio' => '<a>Come stai</a>',
    'data' => date('Y-m-d H:i:s')
];

// Convertilo in JSON e stampalo
echo json_encode($data);
?>
```

Fetch del JSON su Expo:

```js
import React, { useEffect, useState } from 'react';
import { Text, View, ActivityIndicator } from 'react-native';

export default function App() {
  const [messaggio, setMessaggio] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    fetch('https://avabucks.it/app')  // Cambia con il tuo URL
      .then((res) => res.json())
      .then((data) => {
        setMessaggio(data.messaggio);
        setLoading(false);
      })
      .catch((error) => {
        console.error('Errore:', error);
        setMessaggio('Errore nella richiesta');
        setLoading(false);
      });
  }, []);

  return (
    <View style={{ flex: 1, justifyContent: 'center', alignItems: 'center' }}>
      {loading ? <ActivityIndicator size="large" color="blue" /> : <Text>{messaggio}</Text>}
    </View>
  );
}
```

## Push Notification
Installa la dipendenza

```bash
npx expo install expo-notifications expo-device
```

Aggiungi su app.json

```json
{
  "expo": {
    "android": {
      "useNextNotificationsApi": true
    }
  }
}
```

Codice PHP per mandare la notifica all app con token APP_TOKEN:

```php
<?php
function sendExpoNotification($expoPushToken, $title, $message) {
    $url = 'https://exp.host/--/api/v2/push/send';

    $data = [
        'to' => $expoPushToken,
        'title' => $title,
        'body' => $message,
        'sound' => 'default',
        // 'data' => ['extraData' => 'value'], // opzionale
    ];

    $headers = [
        'Accept: application/json',
        'Content-Type: application/json',
    ];

    $ch = curl_init($url);

    curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
    curl_setopt($ch, CURLOPT_POSTFIELDS, json_encode($data));
    curl_setopt($ch, CURLOPT_HTTPHEADER, $headers);

    $response = curl_exec($ch);

    if (curl_errno($ch)) {
        echo 'Errore richiesta: ' . curl_error($ch);
    } else {
        echo 'Risposta: ' . $response;
    }

    curl_close($ch);
}

// Esempio di utilizzo
$token = 'ExponentPushToken[APP_TOKEN]';
sendExpoNotification($token, 'Titolo Notifica', 'Questo è il messaggio!');

?>
```

Funzione per ricevere la notifica su Expo:

```js
import React, { useState, useEffect } from 'react';
import { Text, View, Button, Alert } from 'react-native';
import * as Notifications from 'expo-notifications';
import * as Device from 'expo-device';

// Configure how notifications behave when received in foreground
Notifications.setNotificationHandler({
  handleNotification: async () => ({
    shouldShowAlert: true,
    shouldPlaySound: false,
    shouldSetBadge: false,
  }),
});

export default function App() {
  const [expoPushToken, setExpoPushToken] = useState('');

  async function registerForPushNotificationsAsync() {
    if (Device.isDevice) {
      const { status: existingStatus } = await Notifications.getPermissionsAsync();
      let finalStatus = existingStatus;

      if (existingStatus !== 'granted') {
        const { status } = await Notifications.requestPermissionsAsync();
        finalStatus = status;
      }

      if (finalStatus !== 'granted') {
        Alert.alert('Permesso per le notifiche non concesso!');
        return;
      }

      const tokenData = await Notifications.getExpoPushTokenAsync();
      setExpoPushToken(tokenData.data); // ⬅️ qui lo metti nello state
    } else {
      Alert.alert('Le notifiche push funzionano solo su dispositivi fisici');
    }
  }

  useEffect(() => {
    registerForPushNotificationsAsync();
  }, []);

  return (
    <View style={{ padding: 20, marginTop: 50 }}>
      <Text style={{ marginBottom: 20 }}>Expo Push Token:</Text>
      <Text selectable>{expoPushToken}</Text>
    </View>
  );
}
```
