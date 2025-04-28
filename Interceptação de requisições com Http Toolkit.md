## Interceptação de Requisições em Aplicativos Flutter para Windows e Android com HttpToolkit

Este guia é destinado a projetos Flutter que rodem no Windows e Android. Utilizamos o [HttpToolkit](https://httptoolkit.com/), uma ferramenta que permite visualizar e inspecionar as requisições HTTP/HTTPS feitas pelo aplicativo.

---

### ✅ Pré-requisitos

1. Instalar o [HttpToolkit](https://httptoolkit.com/) no seu computador.
2. Certificar-se de que o aplicativo Flutter esteja preparado para interceptar requisições usando `HttpOverrides`.

---

### 📱 Interceptando requisições no Android

1. Instale o app **HttpToolkit para Android** via Play Store.
2. Conecte o dispositivo Android ao computador e ative a **Depuração USB** nas opções de desenvolvedor.

   ![Image](https://github.com/user-attachments/assets/a54580a8-c2e8-4526-ab78-beea802447da)

3. No HttpToolkit (Windows):
   - Clique em **Intercept** no menu lateral.
   - Selecione **Android device via ADB**.

   ![Image](https://github.com/user-attachments/assets/5eb3491f-ca77-433b-83f7-00af90cc124d)

4. Abra o app HttpToolkit no Android e siga as instruções para instalar o certificado de segurança.

| |  |
| --- | --- |
|![Image](https://github.com/user-attachments/assets/8d6bb145-eaff-47c5-ae98-80a69c4e504e)| ![Image](https://github.com/user-attachments/assets/1fe0d249-57a3-4713-9e7a-8d32222e1c2e)|
![Image](https://github.com/user-attachments/assets/a9bba113-7aa4-435f-a59f-a3031cd7b0df)|![Image](https://github.com/user-attachments/assets/db18461b-9b11-4e6d-9799-fd9fda87b766)|

5. Execute seu aplicativo Android e realize as requisições que deseja interceptar; elas aparecerão em tempo real no painel do HttpToolkit no menu **View**.

  ![Image](https://github.com/user-attachments/assets/e71b662a-0551-4b19-840c-9fc7e84ad948)

---

### 💻 Interceptando requisições no Windows

1. Abra o HttpToolkit no Windows.
2. Execute seu aplicativo Flutter no Windows normalmente.
3. No HttpToolkit, clique em **Intercept** e selecione **Anything**.

   ![Image](https://github.com/user-attachments/assets/f8977b61-ead5-44e7-84e7-da4c8b977c79)


4. Na seção **Step 2: Trust the certificate authority**:
   - Clique em **Export CA certificate**.
   - Instale o certificado gerado no seu sistema.

   ![Image](https://github.com/user-attachments/assets/57d5a0c1-4509-454f-8107-54818359ed5b)
   ![Image](https://github.com/user-attachments/assets/a057e76b-445e-4160-b096-d4f22ebce0ec)


   > ⚡ *Observação:*  
   > A exportação e instalação do certificado é necessária para interceptar tráfego HTTPS corretamente. O HttpToolkit cria uma Autoridade Certificadora (CA) local e utiliza certificados dessa CA para descriptografar conexões HTTPS interceptadas.

5. Realize as requisições que deseja monitorar; elas aparecerão em tempo real no painel do HttpToolkit no menu **View**.

---

### 🛠️ Código de Configuração no Flutter

Para permitir a interceptação, é necessário sobrescrever o comportamento padrão das conexões HTTP no seu aplicativo. Veja como:

#### Classe de Sobrescrição

```dart
import 'dart:io';
import 'package:flutter/foundation.dart';

class AppHttpOverrides extends HttpOverrides {
  @override
  HttpClient createHttpClient(SecurityContext? context) {
    final client = super.createHttpClient(context);

    if (!kDebugMode) return client; // Só ativa em modo debug

    if (Platform.isWindows) {
      client.findProxy = (uri) => 'PROXY localhost:8000'; // Direciona para proxy local no Windows
    }

    client.badCertificateCallback = (X509Certificate cert, String host, int port) => true; // Aceita certificados inválidos

    return client;
  }
}
```

##### Explicação do Código

- **`createHttpClient`**: Cria um novo `HttpClient`, baseando-se no padrão do Flutter.
- **`!kDebugMode`**: Garante que o proxy só será utilizado durante o desenvolvimento, nunca em produção.
- **`Platform.isWindows`**: Verifica se o aplicativo está rodando no Windows.
- **`client.findProxy = (uri) => 'PROXY localhost:8000';`**: Redireciona todas as requisições para o proxy local configurado pelo HttpToolkit, apenas no Windows.
- **`badCertificateCallback`**: Permite que o `HttpClient` aceite qualquer certificado SSL, mesmo que inválido (necessário para interceptação).

#### Inicialização no main.dart

```dart
Future<void> main() async {
  HttpOverrides.global = AppHttpOverrides();
  runApp(AppWidget());
}
```

##### Explicação da Inicialização

- **`HttpOverrides.global = AppHttpOverrides();`**: Define a sobrescrição global para todas as requisições HTTP do aplicativo.
- **`runApp(AppWidget());`**: Inicializa o aplicativo normalmente.

---

## Conclusão

Com essa configuração, você consegue visualizar, monitorar e depurar todas as requisições feitas pelo seu app Flutter no Windows e Android de forma rápida e eficiente, inclusive tráfego HTTPS.

