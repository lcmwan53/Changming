# Introduction to i18n with ngx-translate

A new feature is added: switching languages under page Settings, card General.

To include translations on the front end, we use **ngx-translate** to let users switch languages dynamically without refreshing the web page. The basic idea is to replace the original text with pipe translate, which specifies the original text and a key to finding the translation, and write the corresponding translation in the file **/client/src/assets/i18n/zh-hans.json** and **en-us.json** (and potentially other translation JSON files). There are several scenarios that we may encounter during this process:

## Text in HTML Files

Suppose we are trying to translate  **/client/src/app/auth/login/login.component.html**, we see the following code:

**Original client/src/app/auth/login/login.component.html**
```html
<div style="text-align:center">
  <h2>Log In</h2>
  <div *ngIf="!authService.isLoggedIn()">
  ...
```

First, to make the translation pipe be recognized, we need to import **TranslateModule** in **auth.module.ts**. Then we replace the raw text ‘Log In’ with the **translate** pipe:

**Modified /client/src/app/auth/login/login.component.html**
```html
<div style="text-align:center">
  <h2>{{"auth.login.title.title" | translate: {"raw": "Log In"} }}</h2>
  <div *ngIf="!authService.isLoggedIn()">
  ...
```

Here, a unique key “auth.login.title.title” will be used to find the translations. This key will appear in **en-us.json** and **zh-hans.json**. After that, a key-value combination is passed to the pipe, indicating what the raw text (text in English) should be (in this case, it’s “Log In”). Following such manner, we prepare en-us.json and zh-hans.json as follows:

**en-us.json**
```json
  "auth": {
    "login": {
      "title": {
        "title": "{{ raw }}",
```

**zh-hans.json**
```json
  "auth": {
    "login": {
      "title": {
        "title": "登录",
```

When the language is English, **ngx-translate** will fetch the translation from en-us.json, which is “{{ raw }}”. Subsequently, this format is recognized as a variable and interpolated according to the key-value combination that we passed in the HTML file. Thus, “Log In” will be displayed on the screen. On the other hand, if we switch to Chinese, the value “登录” will be fetched and displayed. In this way, we don’t need to and shouldn’t include any raw text in **en-us.json**, since all values in **en-us.json** need to be in variable format so that they can be interpolated. 

## Text in .TS Files

We can see how translations in .ts files are implemented by this modified **oncoai-patient-detail.components.ts _(only showing parts related to translations)_**:

**oncoai-patient-detail.components.ts**
```typescript
import { TranslateService } from '@ngx-translate/core';
import { merge, of } from 'rxjs';
import { tap } from 'rxjs/operators';
export class OncoaiPatientDetailComponent implements OnInit {


  // for i18n
  displayedNamesRaw = {
    brainTumor: 'Brain Tumor',
    lymphNodes: 'Lymph Node',
    lungNodules: 'Lung Nodule',
    tumor: 'Tumor',
    lesion: 'lesion',
    center: 'Center',
    shortAxis: 'Short Axis',
    longAxis: 'Long Axis'
  }
  displayedNames = Object.assign({}, this.displayedNamesRaw)


  supportedConditions: string[][] = [
    [this.displayedNames.brainTumor, 'braintumor'],
    [this.displayedNames.lymphNodes, 'lymphnodes'],
    [this.displayedNames.lungNodules, 'lungnodules']
  ];
  annotationNameGuide = {
    'roi': [this.displayedNames.lymphNodes, this.displayedNames.tumor, this.displayedNames.lungNodules, this.displayedNames.lesion],
    'points': [this.displayedNames.center],
    'lines': [this.displayedNames.shortAxis, this.displayedNames.longAxis]
  };
 
  constructor(
    private translateService: TranslateService,
  ) {}


  ngOnInit(): void {
    merge(of(null), this.translateService.onLangChange).pipe(
      tap(() => this.getTranslation())
    ).subscribe();
  }


  getTranslation(): void {
    this.translateService.get(
      ['oncoai.oncoaiPatientDetail.supportedConditions.lymphNodes',
      'oncoai.oncoaiPatientDetail.supportedConditions.brainTumor',
      'oncoai.oncoaiPatientDetail.supportedConditions.lungNodules',
      'oncoai.oncoaiPatientDetail.annotationNameGuide.roi.lymphNode',
      'oncoai.oncoaiPatientDetail.annotationNameGuide.roi.tumor',
      'oncoai.oncoaiPatientDetail.annotationNameGuide.roi.lungNodule',
      'oncoai.oncoaiPatientDetail.annotationNameGuide.roi.lesion',
      'oncoai.oncoaiPatientDetail.annotationNameGuide.points.center',
      'oncoai.oncoaiPatientDetail.annotationNameGuide.lines.shortAxis',
      'oncoai.oncoaiPatientDetail.annotationNameGuide.lines.longAxis',
    ], {
      ...this.displayedNamesRaw
    }).subscribe(
        translations => {
          this.supportedConditions = [
            [translations['oncoai.oncoaiPatientDetail.supportedConditions.brainTumor'], 'braintumor'],
            [translations['oncoai.oncoaiPatientDetail.supportedConditions.lymphNodes'], 'lymphnodes'],
            [translations['oncoai.oncoaiPatientDetail.supportedConditions.lungNodules'], 'lungnodules']
          ];
          this.annotationNameGuide = {
            'roi': [translations['oncoai.oncoaiPatientDetail.annotationNameGuide.roi.lymphNode'],
            translations['oncoai.oncoaiPatientDetail.annotationNameGuide.roi.tumor'],
            translations['oncoai.oncoaiPatientDetail.annotationNameGuide.roi.lungNodule'],
            translations['oncoai.oncoaiPatientDetail.annotationNameGuide.roi.lesion']
          ],
          'points': [translations['oncoai.oncoaiPatientDetail.annotationNameGuide.points.center']],
          'lines': [translations['oncoai.oncoaiPatientDetail.annotationNameGuide.lines.shortAxis'],
          translations['oncoai.oncoaiPatientDetail.annotationNameGuide.lines.longAxis']]
          };
        }
      );
  }
}
```
Explanations:

```typescript
import { TranslateService } from '@ngx-translate/core';
import { merge, of } from 'rxjs';
import { tap } from 'rxjs/operators';
```

The above shows the newly imported modules. **TranslateService** for translations, and **merge**, **of** and **tap** to ensure the translations are fetched in real-time.

```typescript
  displayedNamesRaw = {
    brainTumor: 'Brain Tumor',
    lymphNodes: 'Lymph Node',
    lungNodules: 'Lung Nodule',
    tumor: 'Tumor',
    lesion: 'lesion',
    center: 'Center',
    shortAxis: 'Short Axis',
    longAxis: 'Long Axis'
  }
  displayedNames = Object.assign({}, this.displayedNamesRaw)


  supportedConditions: string[][] = [
    [this.displayedNames.brainTumor, 'braintumor'],
    [this.displayedNames.lymphNodes, 'lymphnodes'],
    [this.displayedNames.lungNodules, 'lungnodules']
  ];
  annotationNameGuide = {
    'roi': [this.displayedNames.lymphNodes, this.displayedNames.tumor, this.displayedNames.lungNodules, this.displayedNames.lesion],
    'points': [this.displayedNames.center],
    'lines': [this.displayedNames.shortAxis, this.displayedNames.longAxis]
  };
```

The text in variables _displayedNames_ and _annotationNameGuide_ are replaced by variables, since it’s much more convenient to handle translations from variables. _displayedNamesRaw_ is an object that serves as a dictionary for all possible text in this .ts file. The raw text, showing as the values in this object, should always be in English. _displayedNames_ is a dummy variable, its values are initially copied from _displayedNamesRaw_ and changed real-time every time a new language is set.  

```typescript
    merge(of(null), this.translateService.onLangChange).pipe(
      tap(() => this.getTranslation())
    ).subscribe();
```

This is to make sure the function _getTranslation()_ is called whenever the language change event triggers.

```typescript
getTranslation(): void {
    this.translateService.get(
      ['oncoai.oncoaiPatientDetail.supportedConditions.lymphNodes',
      'oncoai.oncoaiPatientDetail.supportedConditions.brainTumor',
      'oncoai.oncoaiPatientDetail.supportedConditions.lungNodules',
      'oncoai.oncoaiPatientDetail.annotationNameGuide.roi.lymphNode',
      'oncoai.oncoaiPatientDetail.annotationNameGuide.roi.tumor',
      'oncoai.oncoaiPatientDetail.annotationNameGuide.roi.lungNodule',
      'oncoai.oncoaiPatientDetail.annotationNameGuide.roi.lesion',
      'oncoai.oncoaiPatientDetail.annotationNameGuide.points.center',
      'oncoai.oncoaiPatientDetail.annotationNameGuide.lines.shortAxis',
      'oncoai.oncoaiPatientDetail.annotationNameGuide.lines.longAxis',
    ], {
      ...this.displayedNamesRaw
    }).subscribe(
        translations => {
          this.supportedConditions = [
            [translations['oncoai.oncoaiPatientDetail.supportedConditions.brainTumor'], 'braintumor'],
            [translations['oncoai.oncoaiPatientDetail.supportedConditions.lymphNodes'], 'lymphnodes'],
            [translations['oncoai.oncoaiPatientDetail.supportedConditions.lungNodules'], 'lungnodules']
          ];
          this.annotationNameGuide = {
            'roi': [translations['oncoai.oncoaiPatientDetail.annotationNameGuide.roi.lymphNode'],
            translations['oncoai.oncoaiPatientDetail.annotationNameGuide.roi.tumor'],
            translations['oncoai.oncoaiPatientDetail.annotationNameGuide.roi.lungNodule'],
            translations['oncoai.oncoaiPatientDetail.annotationNameGuide.roi.lesion']
          ],
          'points': [translations['oncoai.oncoaiPatientDetail.annotationNameGuide.points.center']],
          'lines': [translations['oncoai.oncoaiPatientDetail.annotationNameGuide.lines.shortAxis'],
          translations['oncoai.oncoaiPatientDetail.annotationNameGuide.lines.longAxis']]
          };
        }
      );
  }
```

The function _getTranslation()_ is used to fetch translations and update the corresponding variables in the class. We provide the translation keys as the first parameter passed to _get()_ (very similar to the keys in the .html part), and specify the key-value combinations for interpolation as the second parameter. In this case, the second parameter is the dictionary _this.displayedNamesRaw_ containing raw text that we defined at the beginning of the file. Finally, after we get the translations from the service, we update the variables with new text. The corresponding **en-us.json** and **zh-hans.json** are shown below:

**en-us.json**
```json
  "oncoai": {
    "oncoaiPatientDetail": {
      "supportedConditions": {
        "brainTumor": "{{ brainTumor }}",
        "lymphNodes": "{{ lymphNodes }}",
        "lungNodules": "{{ lungNodules }}"
      },
      "annotationNameGuide": {
        "roi": {
          "lymphNode": "{{ lymphNodes }}",
          "tumor": "{{ tumor }}",
          "lungNodule": "{{ lungNodules }}",
          "lesion": "{{ lesion }}"
        },
        "points": {
          "center": " {{ center }}"
        },
        "lines": {
          "shortAxis": "{{ shortAxis }}",
          "longAxis": "{{ longAxis }}"
        }
      }
    }
  },
```

**zh-hans.json**

```json
  "oncoai": {
    "oncoaiPatientDetail": {
      "supportedConditions": {
        "brainTumor": "脑肿瘤",
        "lymphNodes": "淋巴结",
        "lungNodules": "肺结节"
      },
      "annotationNameGuide": {
        "roi": {
          "lymphNode": "淋巴结",
          "tumor": "脑肿瘤",
          "lungNodule": "肺结节",
          "lesion": "损伤"
        },
        "points": {
          "center": "居中"
        },
        "lines": {
          "shortAxis": "短轴",
          "longAxis": "长轴"
        }
      }
    }
  },
```
