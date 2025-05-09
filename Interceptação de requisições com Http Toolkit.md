## Interceptação de Requisições em Aplicativos Flutter para Windows e Android com HttpToolkit

Este guia é destinado a projetos Flutter que rodem no Windows e Android. Utilizamos o [HTTP Toolkit](https://httptoolkit.com/), uma ferramenta que permite visualizar e inspecionar as requisições HTTP/HTTPS feitas pelo aplicativo.

---

### ✅ Pré-requisitos

1. Instalar o [HTTP Toolkit](https://httptoolkit.com/) no seu computador.
2. Certificar-se de que o aplicativo Flutter esteja preparado para interceptar requisições usando `HttpOverrides`.

---

### 📱 Interceptando requisições no Android

1. Instale o app **HTTP Toolkit para Android** via Play Store.
2. Conecte o dispositivo Android ao computador e ative a **Depuração USB** nas opções de desenvolvedor.

   ![Image](https://i.ibb.co/355cTFBn/image.png)

3. No HTTP Toolkit (Windows):
   - Clique em **Intercept** no menu lateral.
   - Selecione **Android device via ADB**.

   ![Image](https://i.ibb.co/zHQt2zmf/image.png)

4. Abra o app HttpToolkit no Android e siga as instruções para instalar o certificado de segurança.

| |  |
| --- | --- |
| ![Image](https://i.ibb.co/XZVy32Qr/image.png) | ![Image](https://i.ibb.co/yB58jYwH/image.png) |
|![Image](https://i.ibb.co/pBL3fnBm/image.png)  | ![Image](https://i.ibb.co/8LPj9X1q/image.png) |

5. Execute seu aplicativo Android e realize as requisições que deseja interceptar; elas aparecerão em tempo real no painel do HTTP Toolkit no menu **View**.

  ![Image](https://i.ibb.co/WvRcBkg9/image.png)

---

### 💻 Interceptando requisições no Windows

1. Abra o HTTP Toolkit no Windows.
2. Execute seu aplicativo Flutter no Windows normalmente.
3. No HTTP Toolkit, clique em **Intercept** e selecione **Anything**.

   ![Image](https://i.ibb.co/8pXgDXG/image.png)


4. Na seção **Step 2: Trust the certificate authority**:
   - Clique em **Export CA certificate**.
   - Instale o certificado gerado no seu sistema.

   ![Image](https://i.ibb.co/d0JCdCtQ/image.png)
   ![Image](https://i.ibb.co/Tqhh3rB6/image.png)


   > ⚡ *Observação:*  
   > A exportação e instalação do certificado é necessária para interceptar tráfego HTTPS corretamente. O HTTP Toolkit cria uma Autoridade Certificadora (CA) local e utiliza certificados dessa CA para descriptografar conexões HTTPS interceptadas.

5. Realize as requisições que deseja monitorar; elas aparecerão em tempo real no painel do HTTP Toolkit no menu **View**.

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
