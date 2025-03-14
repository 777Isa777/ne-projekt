import json
import time

import requests
import base64
from PIL import Image
from io import BytesIO

class Text2ImageAPI:

    def __init__(self, url, api_key, secret_key):
        self.URL = url
        self.AUTH_HEADERS = {
            'X-Key': f'Key {api_key}',
            'X-Secret': f'Secret {secret_key}',
        }

    def get_model(self):
        response = requests.get(self.URL + 'key/api/v1/models', headers=self.AUTH_HEADERS)
        data = response.json()
        return data[0]['id']

    def generate(self, prompt, model, images=1, width=1024, height=1024):
        params = {
            "type": "GENERATE",
            "numImages": images,
            "width": width,
            "height": height,
            "generateParams": {
                "query": f"{prompt}"
            }
        }

        data = {
            'model_id': (None, model),
            'params': (None, json.dumps(params), 'application/json')
        }
        response = requests.post(self.URL + 'key/api/v1/text2image/run', headers=self.AUTH_HEADERS, files=data)
        data = response.json()
        return data['uuid']

    def check_generation(self, request_id, attempts=10, delay=10):
        while attempts > 0:
            response = requests.get(self.URL + 'key/api/v1/text2image/status/' + request_id, headers=self.AUTH_HEADERS)
            data = response.json()
            if data['status'] == 'DONE':
                return data['images']

            attempts -= 1
            time.sleep(delay)


if __name__ == '__main__':
    api = Text2ImageAPI('https://api-key.fusionbrain.ai/', '50F257942F492D4DCC9E2D7E44D1EFE6', '8EA938EEB146207E698F2D03512E2FCE')
    model_id = api.get_model()
    uuid = api.generate("Пушистый кот в очках", model_id)
    images = api.check_generation(uuid)[0]


    base64_string = images 


    decoded_data = base64.b64decode(base64_string)


    image = Image.open(BytesIO(decoded_data))

    image.show()
    image.save("decoded_image.jpg")  

'50F257942F492D4DCC9E2D7E44D1EFE6', '8EA938EEB146207E698F2D03512E2FCE'

