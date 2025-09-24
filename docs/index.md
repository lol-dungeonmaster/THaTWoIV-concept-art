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

def generate_image(prompt):
  result = client.models.generate_images(
      model=ultra,
      prompt=prompt,
      config=dict(aspect_ratio="16:9", image_size="2k", person_generation="DONT_ALLOW")
  )
  for n, generated_image in enumerate(result.generated_images):
    (image := generated_image.image).save(f"docs/results/scene_{n}.jpg")
    (image := generated_image.image).show()

generate_image(prompt)
```

# At this rate the harvest was shaping up to be the bees knees...

The men of Upper Vales would often travel a day's journey, crossing the Great River by way of the North Bridge. All for one of their best delicacies -- Honey Ale. Rumor is the expert Woodsmen are unwilling to use the sturdier Dwarvish Bridge to the South. Something about their crafts being unwelcome in those parts. Why even the nearby Greenwood Elves have been known to drop by on their way to Dimrill Dale. Though the Elves have never been willing to share what they use the honey for. They always look amused and say quite plainly they'll eat it -- several barrel's that is.

_-- Somewhere in the Vale, where the bee's are blissfully unaware of the adventure this day will bring._


```python
# Generate from base images

def generate_video(image, prompt):
    # Converting the image to bytes
    image_bytes_io = io.BytesIO()
    image.save(image_bytes_io, format=image.format)
    image_bytes = image_bytes_io.getvalue()

    operation = client.models.generate_videos(
        model="veo-3.0-generate-001", # veo-3.0-generate-001, veo-3.0-fast-generate-001
        prompt=prompt,
        image=types.Image(image_bytes=image_bytes, mime_type=image.format),
        config=types.GenerateVideosConfig(
            aspect_ratio="16:9",
            number_of_videos=1))
    
    while not operation.done:
        print("Waiting for video generation to complete...")
        time.sleep(10)
        operation = client.operations.get(operation)

    return operation.result.generated_videos[0]
```

### Imagen-4.0 standard output


```python
south_1 = Image.open("docs/standard/scene_1.jpg")
```

_Facing south towards Middle Vales from NE.Lindórinand. Greenwood Forest on opposite shore. View of Misty Mountains is hidden by tree-line._

[![](standard/scene_1.jpg)](https://raw.githubusercontent.com/lol-dungeonmaster/THaTWoIV-concept-art/main/docs/results/south_1.mp4)


```python
north_1 = Image.open("docs/standard/scene_2.jpg")
```

_Facing north at NW.Lindórinand near the foothill of Misty Mountains and Nîr Linhaith._

![north_1](standard/scene_2.jpg)


```python
south_2 = Image.open("docs/standard/scene_4.jpg")
```

_Facing south at the entrance to Middle Vales from SE.Lindórinand. Greenwood Forest is opposite shore. Here is where Lindórinand becomes Eaves of Fangorn. By starlight you can hear songs of the Elves echoing over the far tree-line. Their minstrels are said to gather for nightly contest along Sîr Limlaith._

![south_2](standard/scene_4.jpg)

### Imagen-4.0 ultra output


```python
# Describe the generated video for veo-3.0

prompt = """A river and forest during morning golden hour. 
Use a wide-angle lens, 16:9 aspect and 1080p resolution. 
The forest undergrowth is thick with flowering wild berries and old growth.  
The river current is rushing past you. There's no driftwood in the river.
The bee's are feeding on nearby wildflowers. 
The sound of Dipper, Warbler, and Thrush bird song can be heard in the forest.
You are walking along the river bank towards the beehive.
Start further away from the beehive."""
```


```python
# Original lacks clouds in hills (veo/imagen have no post-process editing)

east_1 = Image.open("docs/ultra/scene_1.jpg")
generated_video = generate_video(east_1, prompt)
client.files.download(file=generated_video.video)
generated_video.video.save("docs/results/east_1.mp4")
```


```python
# Edit the original to add clouds

result = client.models.generate_content(
    model="gemini-2.5-flash-image-preview",
    contents=[
        """Add thin clounds to the peaks of only the two mountains the distance. Use studio quality and 16:9 aspect.""", 
        client.files.upload(file="docs/ultra/scene_1.jpg")]
)

for part in result.candidates[0].content.parts:
    if part.text:
        print(part.text)
    elif part.inline_data is not None:
        image = Image.open(BytesIO(part.inline_data.data))
        image.save("docs/results/east_1_edit.png")
```


```python
# Generate video with the edited version

east_1_edit = Image.open("docs/results/east_1_edit.png")
generated_video = generate_video(east_1_edit, prompt)
client.files.download(file=generated_video.video)
generated_video.video.save("docs/results/east_1_edit.mp4")
```

_Facing east along Sîr Ninglor towards confluence with the Great River __(with edits)__. Amon Lanc of Greenwood Mountains can be seen in the distance. NE.Lindórinand on the opposite shore. The absence of marshes is a deliberate device in this version of the legendarium._

[![](results/east_1_edit.png)](https://raw.githubusercontent.com/lol-dungeonmaster/THaTWoIV-concept-art/main/docs/results/east_1_edit.mp4)

_Facing east along Sîr Ninglor towards confluence with the Great River __(original)__. Amon Lanc of Greenwood Mountains can be seen in the distance. NE.Lindórinand on the opposite shore. The absence of marshes is a deliberate device in this version of the legendarium._

[![](ultra/scene_1.jpg)](https://raw.githubusercontent.com/lol-dungeonmaster/THaTWoIV-concept-art/main/docs/results/east_1.mp4)

[![](ultra/scene_1.jpg)](https://raw.githubusercontent.com/lol-dungeonmaster/THaTWoIV-concept-art/main/docs/results/east_2.mp4)

[![](ultra/scene_1.jpg)](https://raw.githubusercontent.com/lol-dungeonmaster/THaTWoIV-concept-art/main/docs/results/east_3.mp4)


```python
# Update imagen prompt for north facing view

prompt = """A 4K studio photo of a forest clearing next to a great river in mid-summer. 
Use a wide-angle lens, golden hour lighting, and 16:9 aspect. 
The river is of large width and it runs straight next to a forested mountain range on the right side. 
The forest is a mixture of fir and pine. The undergrowth is thick with flowering wild berries and old growth. 
Next to the river's edge patches of yellow Iris and blue Myosotis grow along with reeds. 
A rustic beehive can be seen in the forest clearing near the water's edge on the left side.
It's bee's are eagerly feeding on the nearby wildflowers. 
Face the camera across the river width towards the mountains.
With the sun on the right side."""

generate_image(prompt)
```


```python
north_2 = Image.open("docs/ultra/scene_7.jpg")
```

_Facing north along Great River from confluence with Sîr Ninglor. Greenwood Forest and the Forest Road on the opposite shore. The road continues north for another day's journey before turning east. The absence of marshes is a deliberate device in this version of the legendarium._

![north_2](ultra/scene_7.jpg)


```python
# Generate a view of the Forest Road

prompt = """A 4K studio quality photo of the Y-shaped split in an ancient mountain path that travels through a forest.
Use a wide-angle lens, golden hour lighting, and 16:9 aspect. It's mid-summer.
The forest is a mixture of fir and pine. Wildflowers and wildberries grow in the underbrush.
The path is around twelve feet wide. The ground rises to the right of the path. 
To the left and distant down the rise is a wide and straight flowing river.
There is a tall mountain range on the distant right shore of the river.
An even taller mountain range is on the distant left shore of the river and farther away.
The mountain path is made of large and smooth, but rough hewn, tiles of Black Ironstone.
On the left-side of the path is an ancient guard-rail made of stone.
Lichen and Moss grow on the guard-rail and between the tiles of the mountain path.
A stone waypoint marker is located off the mountain path in the underbrush between the left and right branch.
The waypoint marker is smooth granite and shaped like a pyramidal spire with a sun engraving.
The left branch of the split in the path curves downhill and along the river.
The right branch of the split in the path bends right sharply up the hill into a valley."""

generate_image(prompt)
```

_If you use a horse, goat or boar and travel light you can complete the journey in half the time. Which raises a fine point, the dwarves really are marvelous in their road building. Their roads connect the upper reaches of the Vales and beyond to Dimrill Dale. Why there's hardly any hoof-wear or mount fatigue reaching the entrance to the road east from Ninglor Crossing. From here the path splits. The high road leads into the valley and Forest Road. It's another day's journey north on the river-bound road, by pony-reckoning, to Greylin Crossing._

![forest_road_1](ultra/forest_road_1.jpg)


```python
# Generate a view of Celduin Crossing

prompt = """A 4K studio quality photo. Use a wide-angle lens, and 16:9 aspect. It's mid-summer.
A mountain path travels through a forest into a clearing and across a stone bridge.
The bridge crosses a deep and wide river.
The mountain path continues after crossing the bridge into the distant forest.
The mountain path is made of large and smooth, but rough hewn, tiles of Black Ironstone.
The ground rises to the left of the mountain path before the bridge but then flattens into the clearing.
The ground is relatively flat to the right of the mountain path before the bridge and after the bridge.
A tall mountain range rises on the distant horizon.
Stone rubble from an ancient ruin is strewn across the underbrush before crossing the bridge.
Lichen and moss grow on the stone rubble and between the tiles of the mountain path.
The forest on the near-side of the bridge is a mixture of fir and pine.
The forest on the far-side of the bridge is a mixture of blooming linden, beech and hornbeam.
The underbrush is thick with flowering wildberries, wildflowers, old growth.
The path is around twelve feet wide.
The noon-hour sun shines from above-left.
"""

generate_image(prompt)
```

_Two days east by pony, provided you don't get eaten by wild animals, you'll reach Celduin Crossing. This is where the Forest Road exits the Greenwood south of Esgaroth. The bridge is south of Esgaroth-falls where waters from the Forest River and Erebor flow from the long lake. Once, these were are also lands of the Ents. The Eaves of Neldoreth was where Greenwood Forest met the great beech grove. It grew from across the bridge, to the east, past River Carnen. That was before a shadow fell on the east. Now the Ents lament departing those lands, preferring to tend the land south of the bridge._

___"A shadow-lingers-where-green-should-grow that one‑who‑makes‑the‑green‑things‑grow cannot cleanse..."___ _, is their rather cryptic Entish explanation._

_Few, except the Dwarves, use the bridge nowadays. The path leads north along the west side of Esgaroth to their outpost, Erebor. Across the bridge leads north-east to their outpost in Iron Hills. Elves will sometimes traverse the Forest Road then follow the river south-east to Dorwinion. Not to taste the artisan wines, they'll say, so much as the pilgrimage to visit relatives. East-folk from Rhûn occasionally follow River Celduin or Carnen upstream. Those few that reach the Vales are oft-greeted with suspicion by wary Upper Vales-folk._

![forest_road_2](ultra/forest_road_2.jpg)


```python
# Generate a map of the region

map_src = Image.open("docs/src/vales_of_anduin_sketch.jpg")

prompt = """This is a sketch of Vales of Anduin from before S.A 1350. 
The River Anduin's source is upper-left, from Langwell and Greylin. 
It empties into Bay of Belfalas bottom-left after flowing past the island of Tolfalas. 
Generate a new map using old cartography style on borderless parchment. 
The following features must all be included. 
Be accurate in relative placement of features. 
Draw rivers, waterbodies, and waterfalls in blue including Sea of Rhûn. 
Replace forests and mountains with more colorful expressions. 
Place the map title (Vales of Anduin before S.A 1350) over the Harad landmark at bottom-right.

Mountains:
- Ered Mithrin
- Iron Hills
- Greenwood
- Hithaeglir
- Emyn Muil
- Ered Nimrais
- Ered Lithui (north side of Mordor)
- Ephel Dúath (south and west side of Mordor)

Settlements:
- Mount Gundabad
- Dorwinion
- Lindórinand
- Amon Lanc
- Dimrill Dale
- Edhellond

Outposts:
- Erebor
- Iron Hills
- Nindalf
- Tolfalas

River Anduin Tributaries:
- Langwell
- Greylin
- S.Ninglor
- Sîr Celebrant (Dimrill Dale)
- S.Limlaith
- Onodló

Other Rivers:
- Forest River
- Carnen
- Celduin
- Morthond
- Ringló

Waterbodies:
- Esgaroth
- Sea of Rhûn
- Nen Hithoel
- Bay of Belfalas

Waterfalls:
- Erebor's base into Esgaroth
- Esgaroth's base into Celduin
- Dimrill Dale from Hithaeglir into Mirrormere/Celebrant
- Falls of Rauros

Other Landmarks:
- Plateau of Nurn
- Udûn
- Nameless Pass
- Mordor
- Harad

Forests:
- Greenwood Forest on both sides of Greenwood Mountains
- Neldoreth on east side of River Carnen
- Dorwinion on north-west side of Sea of Rhun
- Ambaróna on north side of Emyn Muil and south of Greenwood Forest
- Aldalómë south of S.Limlaith to Ered Nimrais and east to Nen Hithoel
- Anduin Valley Forest from Falls of Rauros south into Bay of Belfalas

Forest Road (dotted pink):
- The east travelling Forest Road heads to Iron Hills
- The north travelling Forest Road heads to Mount Gundabad
- There's a west travelling road from Iron Hills to Erebor and Mount Gundabad
"""

result = client.models.generate_content(
    model="gemini-2.5-flash-image-preview",
    contents=[prompt, map_src]
)

for part in result.candidates[0].content.parts:
    if part.text:
        print(part.text)
    elif part.inline_data is not None:
        image = Image.open(BytesIO(part.inline_data.data))
        image.save("docs/results/Vales_of_Anduin_Map.png")
```

_According a dwarf of dubious sobriety, and a scholarly bard otherwise, the Bay of Belfalas is named thus due to winds created by Anduin Valley. When hot air from the east cools over Ered Mithrin it's funnelled back into the valley from the north. Supposedly it's possible to raft all the way to Bay of Belfalas if you know the river hazards. The only trouble is convincing the Elves to give you a lift home after. Though I'd question the authenticity of this map they left me as proof._

___"Aye, it's real. Many a song tells of rafting the Vales. Not as many for Celduin where travelling by pony is faster --there's no wind at your back, you see. In any case I'm interested in new tales to sing --to reinvent myself as a bard of great jewel-smiths. Keep it. I've no further use for rafting. Mayhaps it will bring you good fortune as it did once for me..."___

_It looks more like a scribble on a napkin than a guide to the Great River._

![map_3](results/Vales_of_Anduin_Map.png)


```python
# Generate a view of Dimrill Dale

prompt = """A 4K studio photo of a peaceful forested dale at the base of a mountain.
The forest is a mixture of fir and pine. The base of the mountain faces east.
Use a wide-angle lens, 16:9 aspect and golden hour lighting.
Face the camera towards the mountain base at 45 degrees.
Morning sunlight shines through the forest canopy above.
On the right side of the mountain base is a stepped waterfall from the mountain.
The waterfall empties into a tear shaped lake. The lake is large with a river at the opposite end.
On the left side of the mountain base and waterfall/lake is a sheer mountain cliff and forest clearing.
High up on the mountain cliff is a rectangular entrance to an underground dwarf kingdom.
A path leads through the forest clearing, along a cliff above the lake.
The path climbs up the cliff to the entrance using a stairway carved into the mountain.
The stepped waterfall can be seen climbing the mountain ever higher.
A path can be seen in the distance following the waterfall's cliff on the same side as the entrance.
Include guard rails and torch-holds along the path.
Be photo-realistic with physically accurate lighting."""

generate_image(prompt)
```

_A bird's eye view of the East-gate into Khazad-dûm in Dimrill Dale. The elves call this place Nanduhirion for the lake, which is said to dim even the light of midday. The Looking-glass Lake, or Mirrormere where you can see your future in the stars of brightest daylight. Or, so the dwarves speak of in tales. Next to the waterfall a mountain pass heads west. Fortunately this is midsummer. In the winter the foot of the mountain pass requires constant snow-removal. Futher east the Mirrormere empties into Sîr Celebrant before confluence with the Great River._

![dimrill_1](ultra/dimrill_1.jpg)


```python
# Generate view of Mirrormere

prompt = """A 4K studio photo of looking into the waters of a lake.
Use a wide-angle lens, 16:9 aspect and golden hour lighting.
The lake is surrounded on all sides by a sheer cliff except opposite the camera is a river.
At the lake's shore is fir and pine forests. The forest continues up on the cliff.
The lake's surface shows a sea of stars.
Close to the camera, some of the stars are connected like a dew-covered spider web in sunlight.
The sky shows clouds and morning daylight. Only the lake shows the stars.
In the lake, the stars that are closer are brighter than the ones farther away.
At the center of the web is also a star.
Make the stars and web look like they're glowing from below the lake surface.
The web appears only faintly between the stars.
The web has stars at major points."""

generate_image(prompt)
```

_I once snuck a peek into the Mirrormere while on consignment for the Ents. "A crown that speaks of good fortune", as the dwarves would boast. I'm not sure I would say I saw a crown. Maybe they meant a crown of stars? After all, this season's honey harvest does look certain to be best ever._

![mirrormere_1](ultra/mirrormere_1.jpg)
