# vue-pdf-worker-fix
vue.js pdf viewer - this is a fork of the original [vue-pdf](https://www.npmjs.com/package/vue-pdf)

## Install
```bash
npm i vue-pdf-worker-fix
```

## Example - basic
```vue
<template>
  <vue-pdf
    src="./static/relativity.pdf"
  />
</template>

<script>
import vuePdf from 'vue-pdf-worker-fix'

export default {
  components: {
    vuePdf
  }
}
```

## Demo

[vue-pdf demo on jsfiddle](https://jsfiddle.net/fossfiddler/5k4ptmqg/145/)

_TBD: fix the demo_

## Browser support
Same browser support as [Vue.js 2](https://github.com/vuejs/vue/blob/dev/README.md)

## Note
since v2.x, the script is exported as esm.

## API

### Props

#### :src <sup>String / Object - default: ''<sup>
The url of the pdf file. `src` may also be a `string|TypedArray|DocumentInitParameters|PDFDataRangeTransport` for more details, see [`PDFJS.getDocument()`](https://github.com/mozilla/pdf.js/blob/8ff1fbe7f819513e7d0023df961e3d223b35aefa/src/display/api.js#L117).

#### :page <sup>Number - default: 1<sup>
The page number to display.

#### :rotate <sup>Number - default: 0<sup>
The page rotation in degrees, only multiple of 90 are valid.

### Events

#### @password <sup>(updatePassword, reason)<sup>
  * `updatePassword`: The function to call with the pdf password.
  * `reason`: the reason why this function is called `'NEED_PASSWORD'` or `'INCORRECT_PASSWORD'`

#### @progress <sup>Number<sup>
Document loading progress. Range [0, 1].

#### @loaded
Triggered when the document is loaded.

#### @page-loaded <sup>Number<sup>
Triggered when a page is loaded.

#### @num-pages <sup>Number<sup>
The total number of pages of the pdf.

#### @error <sup>Object<sup>
Triggered when an error occurred.

#### @link-clicked <sup>Number<sup>
Triggered when an internal link is clicked


### Public methods

#### print(dpi, pageList) * _experimental_ *
  * `dpi`: the print resolution of the document (try 100).
  * `pageList`: the list (array) of pages to print.

### Public static methods

#### createLoadingTask(src[, options])
  * `src`: see `:src` prop  
  * `options`: an object of options. 
  This function creates a PDFJS loading task that can be used and reused as `:src` property.  
  The loading task is a promise that resolves with the PDFJS pdf document that exposes the `numPages` property (see example below).
  
  **beware:** when the component is destroyed, the object returned by `createLoadingTask()` become invalid. 
  
  Supported options:
  * onPassword: Callback that's called when a password protected PDF is being opened.
  * onProgress: Callback return loading progress.
  * withCredentials: Wheter or not to send cookies in the fetch request.


## Examples

##### Example - current page / page count
```vue
<template>
  <div>
    {{currentPage}} / {{pageCount}}
    <vue-pdf
      src="https://cdn.mozilla.net/pdfjs/tracemonkey.pdf"
      @num-pages="pageCount = $event"
      @page-loaded="currentPage = $event"
    />
  </div>
</template>

<script>
import vuePdf from 'vue-pdf-worker-fix'

export default {
  components: {
    vuePdf
  },
  data() {
    return {
      currentPage: 0,
      pageCount: 0,
    }
  }
}

</script>
```


##### Example - display multiple pages of the same pdf document
```vue
<template>
  <div>
    <vue-pdf
      v-for="i in numPages"
      :key="i"
      :src="src"
      :page="i"
      style="display: inline-block; width: 25%"
    />
  </div>
</template>

<script>
import vuePdf from 'vue-pdf-worker-fix'

var loadingTask = pdf.createLoadingTask('https://cdn.mozilla.net/pdfjs/tracemonkey.pdf');

export default {
  components: {
    vuePdf
  },
  data() {
    return {
      src: loadingTask,
      numPages: undefined,
    }
  },
  mounted() {
    this.src.promise.then(pdf => {
      this.numPages = pdf.numPages;
    });
  }
}

</script>
```


##### Example - print all pages
```vue
<template>
  <button @click="$refs.myPdfComponent.print()">
    print
  </button>
  <vue-pdf
    ref="myPdfComponent"
    src="https://cdn.mozilla.net/pdfjs/tracemonkey.pdf"
  />
</template>
```


##### Example - print multiple pages
```vue
<template>
  <button @click="$refs.myPdfComponent.print(100, [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14])">
    print
  </button>
  <vue-pdf
    ref="myPdfComponent"
    src="https://cdn.mozilla.net/pdfjs/tracemonkey.pdf"
  />
</template>
```


##### Example - get text content
```vue
<template>
  <div>
    <button @click="logContent">
      log content
    </button>
    <vue-pdf
      ref="myPdfComponent"
      src="https://cdn.mozilla.net/pdfjs/tracemonkey.pdf"
    />
  </div>
</template>

<script>
import vuePdf from 'vue-pdf-worker-fix'

export default {
  components: {
    vuePdf
  },
  methods: {
    logContent() {
      this.$refs.myPdfComponent.pdf.forEachPage(function(page) {

        return page.getTextContent()
        .then(function(content) {

          var text = content.items.map(item => item.str);
          console.log(text);
        })
      });
    }
  }
}

</script>
```


##### Example - complete
```vue
<template>
  <div>
    <input type="checkbox" v-model="show" />
    <select v-model="src" style="width: 30em">
      <option v-for="item in pdfList" :value="item" v-text="item"></option>
    </select>
    <input v-model.number="page" type="number" style="width: 5em" /> /{{numPages}}
    <button @click="rotate += 90">&#x27F3;</button>
    <button @click="rotate -= 90">&#x27F2;</button>
    <button @click="$refs.pdf.print()">print</button>

    <div style="width: 50%">
      <div
        v-if="loadedRatio > 0 && loadedRatio < 1"
        style="background-color: green; color: white; text-align: center"
        :style="{ width: loadedRatio * 100 + '%' }"
      >
        {{ Math.floor(loadedRatio * 100) }}%
      </div>
      <vue-pdf
        v-if="show"
        ref="pdf"
        :src="src"
        :page="page"
        :rotate="rotate"
        @password="password"
        @progress="loadedRatio = $event"
        @error="error"
        @num-pages="numPages = $event"
        @link-clicked="page = $event"
        style="border: 1px solid red"
      />
    </div>
  </div>
</template>
<script>
import vuePdf from 'vue-pdf-worker-fix'

export default {
  components: {
    vuePdf
  },
  data () {
    return {
      show: true,
      pdfList: [
        '',
        'https://cdn.mozilla.net/pdfjs/tracemonkey.pdf',
        'https://cdn.rawgit.com/mozilla/pdf.js/c6e8ca86/test/pdfs/freeculture.pdf',
        'https://cdn.rawgit.com/mozilla/pdf.js/c6e8ca86/test/pdfs/annotation-link-text-popup.pdf',
        'https://cdn.rawgit.com/mozilla/pdf.js/c6e8ca86/test/pdfs/calrgb.pdf',
        'https://cdn.rawgit.com/sayanee/angularjs-pdf/68066e85/example/pdf/relativity.protected.pdf',
        'data:application/pdf;base64,JVBERi0xLjUKJbXtrvsKMyAwIG9iago8PCAvTGVuZ3RoIDQgMCBSCiAgIC9GaWx0ZXIgL0ZsYXRlRGVjb2RlCj4+CnN0cmVhbQp4nE2NuwoCQQxF+/mK+wMbk5lkHl+wIFislmIhPhYEi10Lf9/MVgZCAufmZAkMppJ6+ZLUuFWsM3ZXxvzpFNaMYjEriqpCtbZSBOsDzw0zjqPHZYtTrEmz4eto7/0K54t7GfegOGCBbBdDH3+y2zsMsVERc9SoRkXORqKGJupS6/9OmMIUfgypJL4KZW5kc3RyZWFtCmVuZG9iago0IDAgb2JqCiAgIDEzOAplbmRvYmoKMiAwIG9iago8PAogICAvRXh0R1N0YXRlIDw8CiAgICAgIC9hMCA8PCAvQ0EgMC42MTE5ODcgL2NhIDAuNjExOTg3ID4+CiAgICAgIC9hMSA8PCAvQ0EgMSAvY2EgMSA+PgogICA+Pgo+PgplbmRvYmoKNSAwIG9iago8PCAvVHlwZSAvUGFnZQogICAvUGFyZW50IDEgMCBSCiAgIC9NZWRpYUJveCBbIDAgMCA1OTUuMjc1NTc0IDg0MS44ODk3NzEgXQogICAvQ29udGVudHMgMyAwIFIKICAgL0dyb3VwIDw8CiAgICAgIC9UeXBlIC9Hcm91cAogICAgICAvUyAvVHJhbnNwYXJlbmN5CiAgICAgIC9DUyAvRGV2aWNlUkdCCiAgID4+CiAgIC9SZXNvdXJjZXMgMiAwIFIKPj4KZW5kb2JqCjEgMCBvYmoKPDwgL1R5cGUgL1BhZ2VzCiAgIC9LaWRzIFsgNSAwIFIgXQogICAvQ291bnQgMQo+PgplbmRvYmoKNiAwIG9iago8PCAvQ3JlYXRvciAoY2Fpcm8gMS4xMS4yIChodHRwOi8vY2Fpcm9ncmFwaGljcy5vcmcpKQogICAvUHJvZHVjZXIgKGNhaXJvIDEuMTEuMiAoaHR0cDovL2NhaXJvZ3JhcGhpY3Mub3JnKSkKPj4KZW5kb2JqCjcgMCBvYmoKPDwgL1R5cGUgL0NhdGFsb2cKICAgL1BhZ2VzIDEgMCBSCj4+CmVuZG9iagp4cmVmCjAgOAowMDAwMDAwMDAwIDY1NTM1IGYgCjAwMDAwMDA1ODAgMDAwMDAgbiAKMDAwMDAwMDI1MiAwMDAwMCBuIAowMDAwMDAwMDE1IDAwMDAwIG4gCjAwMDAwMDAyMzAgMDAwMDAgbiAKMDAwMDAwMDM2NiAwMDAwMCBuIAowMDAwMDAwNjQ1IDAwMDAwIG4gCjAwMDAwMDA3NzIgMDAwMDAgbiAKdHJhaWxlcgo8PCAvU2l6ZSA4CiAgIC9Sb290IDcgMCBSCiAgIC9JbmZvIDYgMCBSCj4+CnN0YXJ0eHJlZgo4MjQKJSVFT0YK',
      ],
      src:'',
      loadedRatio: 0,
      page: 1,
      numPages: 0,
      rotate: 0,
    }
  },
  methods: {
    password: function(updatePassword, reason) {
      updatePassword(prompt('password is "test"'));
    },
    error: function(err) {
      console.log(err);
    }
  }
}
</script>
```

## Credits
[Maintainer: Paolo Zanchi](https://www.linkedin.com/in/paolo-zanchi/)

[Original author: <img src="https://www.franck-freiburger.com/FF.png" width="16"> Franck Freiburger](https://www.franck-freiburger.com)
