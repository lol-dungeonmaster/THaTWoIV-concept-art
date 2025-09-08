# Where to begin...


```python
# Setup the environment

%pip install -qU google-genai==1.29.0 google-api-core lmnr[all] Pillow

import os, IPython
from google import genai
from google.api_core import retry, exceptions
from google.genai import types
from io import BytesIO
from lmnr import Laminar
from PIL import Image

class UserSecretsClient:
    @classmethod
    def get_secret(cls, id: str):
        try:
            return os.environ[id]
        except KeyError as e:
            print(f"KeyError: authentication token for {id} is undefined")

GOOGLE_API_KEY = UserSecretsClient().get_secret("GOOGLE_API_KEY")

try:
    Laminar.initialize(project_api_key=UserSecretsClient().get_secret("LMNR_PROJECT_API_KEY"))
except:
    print("Skipping Laminar.initialize()")

client = genai.Client(api_key=GOOGLE_API_KEY)

is_retriable = lambda e: (isinstance(e, genai.errors.APIError) and e.code in {429, 503, 500})
genai.models.Models.generate_images = retry.Retry(
    predicate=is_retriable)(genai.models.Models.generate_images)
genai.models.Models.generate_videos = retry.Retry(
    predicate=is_retriable)(genai.models.Models.generate_videos)
```

    Note: you may need to restart the kernel to use updated packages.
    KeyError: authentication token for LMNR_PROJECT_API_KEY is undefined
    Skipping Laminar.initialize()


# The bee's are particularly busy this time of year...

The little critters are often too busy to notice the comings and goings of their keepers. Especially when completed at the right time -- before 2nd Breakfast. Well then it's hardly a chore what with them off tending to the blooming mouse-leaf.


```python
prompt = """A 4K studio photo of a forest clearing next to a great river in mid-summer. 
Use a wide-angle lens, golden hour lighting, and 16:9 aspect. 
The river is a vast tributary of large width. 
The forest is a mixture of fir and pine. 
The undergrowth is thick with flowering wild berries and old growth. 
Next to the river's edge patches of yellow Iris and blue Myosotis grow along with reeds. 
A rustic beehive can be seen in the forest clearing near the water's edge. 
It's bee's are eagerly feeding on the nearby wildflowers. 
On the other side a mountain range rises in the distance.
Face the camera across the river width.
With the sun on the right side."""

standard = "imagen-4.0-generate-001"
ultra = "imagen-4.0-ultra-generate-001"
```


```python
result = client.models.generate_images(
    model=standard,
    prompt=prompt,
    config=dict(aspect_ratio="16:9", image_size="2k")
)

for generated_image in result.generated_images:
  (image := generated_image.image).save(f"results/scene_{result.generated_images.index(generated_image)}.jpg")
  (image := generated_image.image).show()
```

# At this rate the harvest was shaping up to be the bees knees...

The men of Upper Vales would even brave the river crossing. All for one of their best delicacies -- honey ale. Why even the nearby Greenwood elves have been known to drop by on their way to Dimrill Dale.

_-- Somewhere in the Vale, where the bee's are blissfully unaware of the adventure this day will bring._

#### Imagen-4.0 standard output


```python
Image.open("standard/scene_1.jpg")
```




    
![standard/scene_1.jpg](standard/scene_1.jpg)
    




```python
Image.open("standard/scene_2.jpg")
```




    
![standard/scene_2.jpg](standard/scene_2.jpg)
    




```python
Image.open("standard/scene_3.jpg")
```




    
![standard/scene_3.jpg](standard/scene_3.jpg)
    




```python
Image.open("standard/scene_4.jpg")
```




    
![standard/scene_4.jpg](standard/scene_4.jpg)
    




```python
Image.open("standard/scene_5.jpg")
```




    
![standard/scene_5.jpg](standard/scene_5.jpg)
    




```python
Image.open("standard/scene_6.jpg")
```




    
![standard/scene_6.jpg](standard/scene_6.jpg)
    



#### Imagen-4.0 ultra output


```python
Image.open("ultra/scene_1.jpg")
```




    
![ultra/scene_1.jpg](ultra/scene_1.jpg)
    




```python
Image.open("ultra/scene_2.jpg")
```




    
![ultra/scene_2.jpg](ultra/scene_2.jpg)
    




```python
Image.open("ultra/scene_3.jpg")
```




    
![ultra/scene_3.jpg](ultra/scene_3.jpg)
    




```python
Image.open("ultra/scene_4.jpg")
```




    
![ultra/scene_4.jpg](ultra/scene_4.jpg)
    




```python
Image.open("ultra/scene_5.jpg")
```




    
![ultra/scene_5.jpg](ultra/scene_5.jpg)
    




```python
Image.open("ultra/scene_6.jpg")
```




    
![ultra/scene_6.jpg](ultra/scene_6.jpg)
    


