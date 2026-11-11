# Praktiskais darbs: 3D sejas rekonstrukcija no viena 2D attēla

**Studenta apliecības numurs:** 221RDB068  
**Izmantotā metode:** DECA

## Darba mērķis un metodes izvēle

Praktiskā darba mērķis bija palaist esošu viena attēla 3D sejas rekonstrukcijas metodi, sagatavot ievades attēlus un izvērtēt iegūto ģeometriju. Tika izmantoti divi pašu uzņemti fotoattēli: frontal.jpg pretskatā un profile.jpg aptuveni 90° profilā. Abi attēli tika apstrādāti atsevišķi, neveidojot vienu apvienotu modeli.

Tika izvēlēta DECA oficiālā realizācija (https://github.com/yfeng95/DECA), jo tā no viena attēla iegūst sejas formu, izteiksmi un smalkas virsmas detaļas, kā arī saglabā poligonu režģus OBJ formātā, un tas ļauj apskatīt sejas ģeometriju no dažādiem skatpunktiem. Tika izmantoti iepriekš apmācīti modeļa svari, apmācība no sākuma netika veikta. Tīkla svaros ir apgūtas sakarības starp attēla pazīmēm un sejas parametriem, savukārt izpildes laikā no konkrētā fotoattēla tiek prognozēti tā sejas formas, izteiksmes, pozas un virsmas detaļu parametri.

## Izmantotā vide

Darbs tika veikts Google Colab Linux vidē ar izpildlaika versiju **2025.10**, un DECA tika palaists atsevišķā Python vidē, nevis ar Colab piezīmju grāmatas pamata Python.

| Komponente | Izmantotā vērtība |
|---|---|
| GPU | NVIDIA Tesla T4 |
| GPU atmiņa pēc `nvidia-smi` | 15 360 MiB |
| Colab pamata Python | 3.12.12 |
| DECA vides Python | 3.10.12 |
| PyTorch DECA vidē | 2.5.1+cu124 |
| torchvision DECA vidē | 0.20.1+cu124 |
| PyTorch CUDA būvējums | 12.4 |
| Sistēmas CUDA kompilators | 12.5.82 |
| g++ | 11.4.0 |
| NumPy / SciPy | 1.23.5 / 1.10.1 |
| Kornia / face-alignment | 0.6.12 / 1.3.5 |
| chumpy | 0.70 |

## Projekta faili

- Datorredze_PR5_221RDB068.ipynb - vides sagatavošana, saderības labojumi, ārējo resursu iegūšana, ievades sagatavošana, abu rekonstrukciju palaišana un vizualizācija.
- .gitignore - noteikumi modeļu, personīgo ievažu, ģenerēto 3D failu un pagaidu failu izslēgšanai.
- images/ - paredzētā mape sešiem atlasītajiem rezultātu attēliem, to pievienošana aprakstīta rezultātu sadaļā.

DECA kods tiek iegūts no sākotnējā repozitorija izpildes laikā. Šajā projektā tiek glabāta darba piezīmju grāmata un dokumentēti nepieciešamie labojumi, nepārpublicējot visu DECA repozitoriju. DECA izmanto savu licenci (https://github.com/yfeng95/DECA/blob/a11554ae2a2b0f3998cf1fa94dd4db03babb34a2/LICENSE), kas jāievēro, lietojot sākotnējo projektu.

## Instalēšana Google Colab vidē

Piezīmju grāmatu atver Google Colab un izpildlaika iestatījumos izvēlas **T4 GPU** un **2025.10** versiju, ja tā ir pieejama. Jaunāka izpildlaika saderība šajā darbā nav pārbaudīta. Zemāk dotās komandas paredzētas jaunai Colab sesijai.

Repozitorija iegūšana:

```python
%cd /content
!git clone https://github.com/yfeng95/DECA.git
!git -C /content/DECA rev-parse HEAD
```

Atsevišķas vides izveidošana:

```python
import subprocess
import sys

# Instalē vides sagatavošanas rīku
subprocess.run([sys.executable, "-m", "pip", "install", "uv"], check=True)

# Izveido atsevišķu Python 3.10 vidi ar pip
subprocess.run([
    sys.executable, "-m", "uv", "venv", "--python", "3.10",
    "--seed", "/content/deca-env"
], check=True)
```

PyTorch un pārējo atkarību instalēšana:

```python
import subprocess

# Norāda DECA vides Python
deca_python = "/content/deca-env/bin/python"

# Instalē izmantoto PyTorch un torchvision pāri ar CUDA 12.4 atbalstu
subprocess.run([
    deca_python, "-m", "pip", "install",
    "torch==2.5.1", "torchvision==0.20.1",
    "--index-url", "https://download.pytorch.org/whl/cu124"
], check=True)

# Nosaka pamatatkarību versijas
pakotnes = [
    "torch==2.5.1", "torchvision==0.20.1",
    "numpy==1.23.5", "scipy==1.10.1",
    "scikit-image==0.21.0", "opencv-python==4.8.1.78",
    "matplotlib==3.7.5", "Pillow==10.4.0",
    "PyYAML==6.0.2", "yacs==0.1.8",
    "kornia==0.6.12", "face-alignment==1.3.5",
    "numba==0.60.0", "ninja==1.11.1.3",
    "fvcore==0.1.5.post20221221"
]
subprocess.run([deca_python, "-m", "pip", "install", *pakotnes], check=True)

# Sagatavo vecākās bibliotēkas būvēšanu un instalē chumpy
subprocess.run([
    deca_python, "-m", "pip", "install",
    "setuptools==69.5.1", "wheel==0.45.1"
], check=True)
subprocess.run([
    deca_python, "-m", "pip", "install", "--no-build-isolation", "chumpy==0.70"
], check=True)
subprocess.run([deca_python, "-m", "pip", "check"], check=True)
```

Vizualizācijai tika izmantota Colab pamata vidē pieejamā Plotly bibliotēka. Ja citā Colab vidē tā nav pieejama, tā jāinstalē piezīmju grāmatas pamata vidē, DECA rekonstrukcijas vide un Plotly skatītāja vide ir atsevišķas.

### Saderības labojumi

Pēc klonēšanas un pirms rekonstrukcijas piezīmju grāmatā izpilda abas koda labošanas šūnas. Tās piemēro šādus aizvietojumus:

| Fails | Veiktā izmaiņa |
|---|---|
| `decalib/utils/renderer.py` | `'-std=c++14', '-ccbin=$$(which gcc-7)'` aizstāts ar `'-std=c++17'` |
| `decalib/utils/rasterizer/standard_rasterize_cuda_kernel.cu` | `face_vertices.type()` aizstāts ar `face_vertices.scalar_type()`, bet `.data<` — ar `.data_ptr<` |
| `decalib/datasets/detectors.py` | `face_alignment.LandmarksType._2D` aizstāts ar `face_alignment.LandmarksType.TWO_D` |

Labojumi bija nepieciešami vecā projekta saskarnes pielāgošanai izmantotajai videi. Rekonstrukcijas metode netika aizstāta ar citu realizāciju, un labošanas šūnas paredzētas vienreizējai izpildei sākotnējā kodā, jau piemērota labojuma atkārtošana var izraisīt pārbaudes `AssertionError`.

### FLAME2020 un DECA svari

1. FLAME oficiālajā lejupielāžu lapā (https://flame.is.tue.mpg.de/download.php) izveido kontu vai piesakās, iepazīstas ar licences nosacījumiem un lejupielādē **FLAME 2020**, nevis citu modeļa versiju.
2. Ar piezīmju grāmatas augšupielādes šūnu Colab ievieto `FLAME2020.zip`. Nākamā šūna no arhīva izņem `generic_model.pkl` un saglabā to `/content/DECA/data/generic_model.pkl`.
3. DECA svarus iegūst ar zemāk norādītajām komandām. Faila identifikators ir ņemts no oficiālā projekta fetch_data.sh (https://github.com/yfeng95/DECA/blob/a11554ae2a2b0f3998cf1fa94dd4db03babb34a2/fetch_data.sh).

```python
# Instalē Google Drive lejupielādes rīku DECA vidē
!/content/deca-env/bin/python -m pip install gdown==5.2.0

# Lejupielādē oficiālos DECA svarus paredzētajā vietā
!/content/deca-env/bin/python -m gdown 1rp8kdyLPvErw2dTmqtjISRVvQLj6Yzje -O /content/DECA/data/deca_model.tar
```

`deca_model.tar` netiek atpakots - to ielādē PyTorch. Papildu albedo resurss `FLAME_albedo_from_BFM.npz` nav nepieciešams, jo izmantots `--useTex False`. Pirmajā sejas noteicēja palaišanā tā svari lejupielādējas automātiski, taču pārējie izmantotie ģeometrijas, masku un detalizācijas resursi tiek iegūti kopā ar DECA repozitoriju.

Modeļu svarus, FLAME failu, ZIP arhīvu un oficiālā projekta resursus šī darba repozitorijā neiekļauj.

## Ievades sagatavošana

Piezīmju grāmatā vispirms augšupielādē `frontal.jpg`, saglabā oriģinālu un izveido darba kopiju. Pēc pretskata rekonstrukcijas pārbaudes analogi sagatavo `profile.jpg`.

| Ievade | Oriģināls Colab sesijā | Darba kopija Colab sesijā |
|---|---|---|
| Pretskats | `/content/DECA/inputs/originals/frontal/frontal.jpg` | `/content/DECA/inputs/processed/frontal/frontal.jpg` |
| Profils | `/content/DECA/inputs/originals/profile/profile.jpg` | `/content/DECA/inputs/processed/profile/profile.jpg` |

Oriģināli netika pārveidoti. Darba kopijām tika piemērota EXIF metadatos norādītā orientācija un RGB krāsu formāts, kā arī attēli tika proporcionāli samazināti līdz ne vairāk kā 1600 pikseļiem garākajā malā un saglabāti JPEG formātā ar kvalitātes parametru 95. Abas izmantotās darba kopijas bija **1200 x 1600 pikseļi**. DECA papildus automātiski noteica seju un sagatavoja apgrieztu **224 x 224 pikseļu** ievadi.

## Rekonstrukcijas palaišana

Pilnās palaišanas šūnas piezīmju grāmatā saglabā arī konsoles žurnālus un izvada rezultātu failu sarakstu. Zemāk norādītas tās pašas rekonstrukcijas komandas Colab `%%bash` šūnām. Katrā šūnā jāiekļauj visa attiecīgā komanda kopā ar vides iestatījumiem.

### Pretskata ievade

```bash
%%bash
set -e
cd /content/DECA

# Norāda DECA vides rīkus un Tesla T4 kompilēšanas iestatījumus
export PATH="/content/deca-env/bin:$PATH"
export MAX_JOBS=2
export TORCH_CUDA_ARCH_LIST=7.5

# Rekonstruē tikai pretskata attēlu
/content/deca-env/bin/python demos/demo_reconstruct.py \
  -i /content/DECA/inputs/processed/frontal/frontal.jpg \
  -s /content/DECA/results/frontal \
  --device cuda --iscrop True --rasterizer_type standard \
  --useTex False --render_orig False \
  --saveObj True --saveVis True --saveImages True
```

### Profila ievade

Profila komandu palaiž pēc pretskata rezultāta pārbaudes.

```bash
%%bash
set -e
cd /content/DECA

# Norāda tos pašus vides iestatījumus kā pretskatam
export PATH="/content/deca-env/bin:$PATH"
export MAX_JOBS=2
export TORCH_CUDA_ARCH_LIST=7.5

# Rekonstruē tikai profila attēlu atsevišķā rezultātu mapē
/content/deca-env/bin/python demos/demo_reconstruct.py \
  -i /content/DECA/inputs/processed/profile/profile.jpg \
  -s /content/DECA/results/profile \
  --device cuda --iscrop True --rasterizer_type standard \
  --useTex False --render_orig False \
  --saveObj True --saveVis True --saveImages True
```

| Parametrs | Nozīme |
|---|---|
| `--iscrop True` | Automātiski nosaka un apgriež seju |
| `--rasterizer_type standard` | Izmanto projekta CUDA renderētāju ar dokumentētajiem labojumiem |
| `--useTex False` | Neizmanto papildu albedo modeli |
| `--render_orig False` | Neveido papildu renderu pilnā ievades izšķirtspējā |
| `--saveObj True` | Saglabā pamata un detalizēto OBJ režģi |
| `--saveVis True` | Saglabā kopējo pārskata attēlu |
| `--saveImages True` | Saglabā atsevišķus vizualizāciju attēlus |

## Rezultāti

Abas izpildes beidzās ar **atgriešanās kodu 0**, kā arī žurnālos netika konstatēts paziņojums `no face detected`. Rezultāti papildus tika apskatīti vizuāli Colab vidē.

| Ievade | Pamata režģis | Detalizētais režģis |
|---|---|---|
| `frontal.jpg` | `results/frontal/frontal/frontal.obj` | `results/frontal/frontal/frontal_detail.obj` |
| `profile.jpg` | `results/profile/profile/profile.obj` | `results/profile/profile/profile_detail.obj` |

Ceļi tabulā ir relatīvi pret `/content/DECA` izpildes laikā. Katrā rezultātu mapē piezīmju grāmatas palaišanas šūna saglabāja arī `reconstruction.log`. Tika iegūtas materiālu un tekstūras datnes, sejas apgriezuma, orientieru un ģeometrijas vizualizācijas. `--useTex False` atslēdz papildu albedo modeli, bet neatceļ noklusēto iespēju iegūt tekstūru no ievades attēla.

Detalizētie OBJ modeļi tika atvērti Plotly skatītājā. Abiem tika saglabāts pretskats, profils abos virzienos un trīs ceturtdaļu skats vienā virzienā - **kopā astoņi attēli**. Modeļa pagriešana maina tikai virtuālās kameras skatpunktu un nerada jaunu rekonstrukciju.

### Rezultātu attēlu pievienošana repozitorijam

| Modeļa ievade | Skatpunkts | Paredzētais faila nosaukums |
|---|---|---|
| `frontal.jpg` | Pretskats | `images/frontal_view_front.png` |
| `frontal.jpg` | Kreisais profils | `images/frontal_view_left.png` |
| `frontal.jpg` | Labais profils | `images/frontal_view_right.png` |
| `frontal.jpg` | Trīs ceturtdaļu skats | `images/frontal_view_3quarter.png` |
| `profile.jpg` | Pretskats | `images/profile_view_front.png` |
| `profile.jpg` | Kreisais profils | `images/profile_view_left.png` |
| `profile.jpg` | Labais profils | `images/profile_view_right.png` |
| `profile.jpg` | Trīs ceturtdaļu skats | `images/profile_view_3quarter.png` |

Kreisais un labais profils apzīmē attiecīgo rekonstruētās personas sejas pusi, nevis attēla kreiso vai labo malu. Abu modeļu trīs ceturtdaļu skati tika saglabāti vienā virzienā, lai atvieglotu to salīdzināšanu.

Faila nosaukuma sākums apzīmē rekonstrukcijas ievadi, bet beigas - modeļa skatpunktu, piemēram, profile_view_front.png ir no profila fotoattēla iegūtā modeļa pretskats.

## Rezultātu analīze

Abu ievades attēlu rekonstrukcijās vizuāli šķiet saglabāts sejas platums, deguna kopējā forma, žokļa līnija un zods, redzamas arī krokas pie deguna un vaigiem. Būtiskas deformācijas netika novērotas, taču deguna formai ir nelielas atšķirības. No pretskata iegūtajā modelī acu lielums šķiet labāk saglabāts nekā no profila iegūtajā modelī. Pelēkajā skatītājā redzamais ēnojums ir atkarīgs arī no renderētāja apgaismojuma, tāpēc to nevar uzskatīt par pierādījumu oriģinālā attēla tekstūras saglabāšanai.

Salīdzinot abus modeļus vienādā trīs ceturtdaļu skatā, pretskata rekonstrukcija subjektīvi šķiet līdzīgāka oriģinālajai sejai, jo tajā labāk izteiktas ieloces un iedobes vaigu un deguna apvidū, taču profila rekonstrukcijā šīs detaļas ir mazāk izteiktas, un atšķiras rekonstruētā sejas izteiksme. Atšķirības var būt saistītas ar galvas pozu, redzamās informācijas apjomu un izteiksmes vai apgaismojuma atšķirībām starp ievadēm. Šo iespējamo cēloņu ietekme atsevišķos eksperimentos netika nodalīta.

No viena fotoattēla nav iespējams tieši novērot visu sejas virsmu, jo pretskatā ir ierobežota informācija par sejas dziļumu, savukārt profilā nav redzama pretējā sejas puse. Šo apgabalu ģeometrija ir atkarīga no DECA apmācībā apgūtajām sakarībām un FLAME formas modeļa, tādēļ vizuālā līdzība un vienmērīga modeļa pagriešana neapliecina pareizus fiziskos izmērus. Ģeometriskā precizitāte milimetros šajā darbā netika noteikta.

Neatkarīgai vizuālai pārbaudei varētu uzņemt papildu fotoattēlu aptuveni 45° leņķī ar neitrālu izteiksmi un līdzīgu apgaismojumu, un, neizmantojot šo attēlu rekonstrukcijai, varētu salīdzināt deguna, lūpu un zoda kontūras ar katra modeļa atbilstošo skatpunktu. Šāda pārbaude palīdzētu atklāt formas neatbilstības, taču nenodrošinātu metriskas precizitātes mērījumu. Aprakstītā pārbaude šajā darbā netika veikta.

## Novērstās kļūdas un zināmie ierobežojumi

- **Atkarību konflikts:** sākotnēji izmantotā Kornia 0.4.0 prasīja PyTorch versiju zem 1.7. Konflikts tika novērsts ar Kornia 0.6.12.
- **Sejas noteicēja saskarne:** `LandmarksType._2D` izraisīja `AttributeError`, jo instalētajā face-alignment 1.3.5 ir `TWO_D`. Pēc aizvietojuma abas rekonstrukcijas izdevās.
- **Renderētāja saderība:** tika veikti iepriekš aprakstītie C++ un CUDA izsaukumu labojumi. PyTorch CUDA 12.4 būvējums un sistēmas CUDA kompilators 12.5 šajā izpildē darbojās kopā, citu kombināciju saderība nav pārbaudīta.

## Izmantotie avoti un MI palīdzība

- DECA oficiālais projekts (https://github.com/yfeng95/DECA) un izmantotās versijas demonstrācijas skripts (https://github.com/yfeng95/DECA/blob/a11554ae2a2b0f3998cf1fa94dd4db03babb34a2/demos/demo_reconstruct.py).
- Feng, Y., Feng, H., Black, M. J., Bolkart, T. **Learning an Animatable Detailed 3D Face Model from In-The-Wild Images.** ACM Transactions on Graphics, SIGGRAPH, 2021. Publikācija (https://arxiv.org/abs/2012.04012).
- Li, T., Bolkart, T., Black, M. J., Li, H., Romero, J. **Learning a model of facial shape and expression from 4D scans.** ACM Transactions on Graphics, SIGGRAPH Asia, 2017. FLAME projekta lapa (https://flame.is.tue.mpg.de/).
- PyTorch iepriekšējo versiju instalēšana (https://pytorch.org/get-started/previous-versions/).
- uv vides izveidošana (https://docs.astral.sh/uv/pip/environments/).
- face-alignment projekts (https://github.com/1adrianb/face-alignment).
- Kornia projekts (https://github.com/kornia/kornia).
- gdown lejupielādes rīks (https://github.com/wkentaro/gdown).
- Plotly 3D režģu vizualizācija (https://plotly.com/python/3d-mesh/).

Koda izstrādē, vides konfigurēšanā un saderības kļūdu novēršanā kā palīgrīks tika izmantots MI rīks ChatGPT.
