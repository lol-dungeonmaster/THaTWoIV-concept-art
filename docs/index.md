# Where to begin...


```python
# Setup the environment

%pip install -qU google-genai==1.29.0 google-api-core lmnr[all] Pillow

import os, io, IPython, time
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
genai.models.Models.generate_content = retry.Retry(
    predicate=is_retriable)(genai.models.Models.generate_content)
```

    Note: you may need to restart the kernel to use updated packages.
    KeyError: authentication token for LMNR_PROJECT_API_KEY is undefined
    Skipping Laminar.initialize()


# The bee's are particularly busy this time of year...

The little critters are often too busy to notice the comings and goings of their keepers. Especially when completed at the right time -- before 2nd Breakfast. Well then it's hardly a chore what with them off tending to the blooming mouse-leaf.


```python
# Generate base images

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

result = client.models.generate_images(
    model=standard,
    prompt=prompt,
    config=dict(aspect_ratio="16:9", image_size="2k")
)

for n, generated_image in enumerate(result.generated_images):
  (image := generated_image.image).save(f"docs/results/scene_{n}.jpg")
  (image := generated_image.image).show()
```

# At this rate the harvest was shaping up to be the bees knees...

The men of Upper Vales would often travel a day's journey, crossing the Great River by way of the North Bridge. All for one of their best delicacies -- Honey Ale. Rumor is the expert Woodsmen are unwilling to use the sturdier Dwarvish Bridge to the South. Something about their crafts being unwelcome in those parts. Why even the nearby Greenwood Elves have been known to drop by on their way to Dimrill Dale. Though the Elves have never been willing to share what they use the honey for. They always look amused and say quite plainly they'll eat it -- several barrel's that is.

_-- Somewhere in the Vale, where the bee's are blissfully unaware of the adventure this day will bring._


```python
# Generate from base images

def generate_video(image):
    prompt = """This 4K studio photo depicts a river and forest during morning golden hour.
    Use a wide-angle lens and 16:9 aspect.
    The forest is a mixture of fir and pine. The undergrowth is thick with flowering wild berries and old growth.
    In the forest birds are calling.
    You are walking on the river bank towards the beehive.
    The bee's are feeding on nearby wildflowers.
    Start further away from the beehive.
    The river current is rushing past you.
    Preserve existing lighting and studio quality."""

    # Converting the image to bytes
    image_bytes_io = io.BytesIO()
    image.save(image_bytes_io, format=image.format)
    image_bytes = image_bytes_io.getvalue()

    operation = client.models.generate_videos(
        model="veo-3.0-generate-001",
        prompt=prompt,
        image=types.Image(image_bytes=image_bytes, mime_type=image.format),
    )

    while not operation.done:
        print("Waiting for video generation to complete...")
        time.sleep(10)
        operation = client.operations.get(operation)

    return operation.result.generated_videos[0]
```

#### Imagen-4.0 standard output


```python
south_1 = Image.open("docs/standard/scene_1.jpg")
```

_Facing south towards Middle Vales. Misty Mountains is hidden by tree-line._

![south_1](standard/scene_1.jpg)


```python
north_1 = Image.open("docs/standard/scene_2.jpg")
```

_Facing north near the footsteps of Misty Mountains._

![north_1](standard/scene_2.jpg)


```python
Image.open("docs/standard/scene_3.jpg")
```


![standard/scene_3.jpg](standard/scene_3.jpg)


```python
Image.open("docs/standard/scene_4.jpg")
```


![standard/scene_4.jpg](standard/scene_4.jpg)


```python
Image.open("docs/standard/scene_5.jpg")
```


![standard/scene_5.jpg](standard/scene_5.jpg)


```python
Image.open("docs/standard/scene_6.jpg")
```

![standard/scene_6.jpg](standard/scene_6.jpg)


#### Imagen-4.0 ultra output


```python
east_1 = Image.open("docs/ultra/scene_1.jpg")
generated_video = generate_video(east_1)
client.files.download(file=generated_video.video)
generated_video.video.save("docs/results/east_1.mp4")
```

    Waiting for video generation to complete...
    Waiting for video generation to complete...
    Waiting for video generation to complete...
    Waiting for video generation to complete...
    Waiting for video generation to complete...


_Facing east towards confluence with the Great River. Foothills of Greenwood Mountains in the distance. Lindórinand on the south shore._

[![](ultra/scene_1.jpg)](https://raw.githubusercontent.com/lol-dungeonmaster/THaTWoIV-concept-art/main/docs/results/east_1.mp4)


```python
Image.open("docs/ultra/scene_2.jpg")
```


![ultra/scene_2.jpg](ultra/scene_2.jpg)


```python
Image.open("docs/ultra/scene_3.jpg")
```


![ultra/scene_3.jpg](ultra/scene_3.jpg)


```python
Image.open("docs/ultra/scene_4.jpg")
```


![ultra/scene_4.jpg](ultra/scene_4.jpg)


```python
Image.open("docs/ultra/scene_5.jpg")
```


![ultra/scene_5.jpg](ultra/scene_5.jpg)


```python
Image.open("docs/ultra/scene_6.jpg")
```


![ultra/scene_6.jpg](ultra/scene_6.jpg)

