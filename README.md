# dl-facial-recognition-webcam-12-24
O código apresentado é um script para implementar detecção facial em imagens e vídeos capturados por webcam.
---

# Explicação Detalhada do Código: Reconhecimento Facial com Webcam

## Introdução

O código apresentado é um script que utiliza o Google Colab, a biblioteca OpenCV, e outras dependências para implementar detecção facial em imagens e vídeos capturados por webcam. Ele se baseia no uso de **Haar Cascade Classifier**, um modelo pré-treinado de detecção de objetos disponibilizado pelo OpenCV.

O objetivo principal é capturar imagens ou vídeos da webcam e identificar faces em tempo real, desenhando caixas delimitadoras ao redor dos rostos detectados.

---

## Importação de Bibliotecas e Configurações Iniciais

```python
from IPython.display import display, Javascript, Image
from google.colab.output import eval_js
from google.colab.patches import cv2_imshow
from base64 import b64decode, b64encode
import cv2
import numpy as np
import PIL
import io
import html
import time
import matplotlib.pyplot as plt
```

### Explicação

- **IPython**: Manipula exibições interativas no Google Colab.
- **google.colab.output**: Facilita a comunicação entre o navegador e o Python para capturar imagens da webcam.
- **OpenCV (`cv2`)**: Biblioteca usada para visão computacional, incluindo a detecção facial.
- **PIL e io**: Gerenciam imagens no formato PIL (Python Imaging Library).
- **NumPy**: Manipula arrays numéricos, que representam imagens.
- **Matplotlib**: Permite exibir imagens de forma gráfica.

---

## Funções Auxiliares

### 1. `js_to_image`

Converte uma imagem capturada em JavaScript para o formato BGR do OpenCV.

```python
def js_to_image(js_reply):
    """
    Params:
        js_reply: Objeto JavaScript contendo a imagem da webcam.
    Returns:
        img: Imagem no formato OpenCV (BGR).
    """
    image_bytes = b64decode(js_reply.split(',')[1])
    jpg_as_np = np.frombuffer(image_bytes, dtype=np.uint8)
    img = cv2.imdecode(jpg_as_np, flags=1)
    return img
```

#### Explicação

- Decodifica uma string de imagem base64 recebida do navegador.
- Converte os bytes para um array NumPy e, em seguida, para o formato BGR, usado pelo OpenCV.

---

### 2. `bbox_to_bytes`

Gera uma imagem base64 a partir de um array NumPy contendo uma sobreposição de caixas delimitadoras.

```python
def bbox_to_bytes(bbox_array):
    """
    Params:
        bbox_array: Array NumPy contendo os pixels da caixa delimitadora.
    Returns:
        Base64: String de imagem no formato base64.
    """
    bbox_PIL = PIL.Image.fromarray(bbox_array, 'RGBA')
    iobuf = io.BytesIO()
    bbox_PIL.save(iobuf, format='png')
    bbox_bytes = 'data:image/png;base64,{}'.format((str(b64encode(iobuf.getvalue()), 'utf-8')))
    return bbox_bytes
```

#### Explicação

- Converte um array NumPy para uma imagem no formato PIL.
- Codifica a imagem no formato base64 para exibição no navegador.

---

## Carregamento do Classificador Haar Cascade

```python
face_cascade = cv2.CascadeClassifier(cv2.samples.findFile(cv2.data.haarcascades + 'haarcascade_frontalface_default.xml'))
```

#### Explicação

- O modelo Haar Cascade pré-treinado para detecção de rostos é carregado da biblioteca OpenCV.

---

## Captura de Imagem da Webcam

### Função: `take_photo`

Captura uma imagem da webcam e detecta rostos nela.

```python
def take_photo(filename='photo.jpg', quality=0.8):
    js = Javascript('''
        async function takePhoto(quality) {
            const div = document.createElement('div');
            const capture = document.createElement('button');
            capture.textContent = 'Capture';
            div.appendChild(capture);
            const video = document.createElement('video');
            video.style.display = 'block';
            const stream = await navigator.mediaDevices.getUserMedia({video: true});
            document.body.appendChild(div);
            div.appendChild(video);
            video.srcObject = stream;
            await video.play();
            google.colab.output.setIframeHeight(document.documentElement.scrollHeight, true);
            await new Promise((resolve) => capture.onclick = resolve);
            const canvas = document.createElement('canvas');
            canvas.width = video.videoWidth;
            canvas.height = video.videoHeight;
            canvas.getContext('2d').drawImage(video, 0, 0);
            stream.getVideoTracks()[0].stop();
            div.remove();
            return canvas.toDataURL('image/jpeg', quality);
        }
    ''')
    display(js)
    data = eval_js('takePhoto({})'.format(quality))
    img = js_to_image(data)
    gray = cv2.cvtColor(img, cv2.COLOR_RGB2GRAY)
    faces = face_cascade.detectMultiScale(gray)
    for (x, y, w, h) in faces:
        img = cv2.rectangle(img, (x, y), (x+w, y+h), (255, 0, 0), 2)
    cv2.imwrite(filename, img)
    return filename
```

#### Explicação

1. Um botão interativo no navegador é gerado para capturar a imagem.
2. Após a captura:
   - A imagem é convertida para o formato OpenCV.
   - Convertida para escala de cinza para detecção de rostos.
3. Os rostos são detectados e caixas delimitadoras são desenhadas ao redor de cada rosto.
4. A imagem resultante é salva no arquivo especificado.

---

## Captura de Vídeo da Webcam

O trecho para vídeos estende a lógica de imagens, processando quadro a quadro em tempo real e sobrepondo caixas delimitadoras.

---

## Execução do Código

```python
try:
    filename = take_photo('photo.jpg')
    print('Saved to {}'.format(filename))
    display(Image(filename))
except Exception as err:
    print(str(err))
```

#### Explicação

- A função `take_photo` é chamada para capturar e processar uma imagem.
- A imagem processada é salva e exibida.
- Caso ocorra um erro, ele é exibido (exemplo: webcam não disponível).

---

## Conclusão

Este código exemplifica como integrar OpenCV com o Google Colab para realizar tarefas de visão computacional. Ele utiliza técnicas fundamentais de processamento de imagens e manipulação interativa no navegador para capturar, processar e exibir imagens e vídeos da webcam com detecção facial.
