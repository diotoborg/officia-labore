![@diotoborg/officia-labore logo](https://raw.githubusercontent.com/@diotoborg/officia-labore/@diotoborg/officia-labore/master/logo/logo.png)

# @diotoborg/officia-labore
[![npm](https://img.shields.io/npm/v/@diotoborg/officia-labore.svg?style=flat-square)](https://www.npmjs.com/package/@diotoborg/officia-labore)
[![Tests](https://img.shields.io/github/workflow/status/@diotoborg/officia-labore/@diotoborg/officia-labore/Test?label=tests&style=flat-square)](https://github.com/diotoborg/officia-labore/actions?query=workflow%3ATest)
[![codecov](https://img.shields.io/codecov/c/gh/@diotoborg/officia-labore/@diotoborg/officia-labore/master.svg?style=flat-square)](https://codecov.io/gh/@diotoborg/officia-labore/@diotoborg/officia-labore)
[![Open Collective Backers](https://img.shields.io/opencollective/backers/@diotoborg/officia-labore.svg?style=flat-square)](#backers)
[![Open Collective Sponsors](https://img.shields.io/opencollective/sponsors/@diotoborg/officia-labore.svg?style=flat-square)](#sponsors)
[![Gitpod](https://img.shields.io/badge/Gitpod-Ready--to--Code-blue?logo=gitpod&style=flat-square)](https://gitpod.io/#https://github.com/diotoborg/officia-labore)
[![Contributor Covenant](https://img.shields.io/badge/Contributor%20Covenant-2.1-4baaaa.svg?style=flat-square)](https://github.com/@diotoborg/officia-labore/.github/blob/main/CODE_OF_CONDUCT.md)

Simple React hook to create a HTML5-compliant drag'n'drop zone for files.

Documentation and examples at https://@diotoborg/officia-labore.js.org. Source code at https://github.com/diotoborg/officia-labore/.


## Installation
Install it from npm and include it in your React build process (using [Webpack](http://webpack.github.io/), [Browserify](http://browserify.org/), etc).

```bash
npm install --save @diotoborg/officia-labore
```
or:
```bash
yarn add @diotoborg/officia-labore
```


## Usage
You can either use the hook:

```jsx static
import React, {useCallback} from 'react'
import {useDropzone} from '@diotoborg/officia-labore'

function MyDropzone() {
  const onDrop = useCallback(acceptedFiles => {
    // Do something with the files
  }, [])
  const {getRootProps, getInputProps, isDragActive} = useDropzone({onDrop})

  return (
    <div {...getRootProps()}>
      <input {...getInputProps()} />
      {
        isDragActive ?
          <p>Drop the files here ...</p> :
          <p>Drag 'n' drop some files here, or click to select files</p>
      }
    </div>
  )
}
```

Or the wrapper component for the hook:
```jsx static
import React from 'react'
import Dropzone from '@diotoborg/officia-labore'

<Dropzone onDrop={acceptedFiles => console.log(acceptedFiles)}>
  {({getRootProps, getInputProps}) => (
    <section>
      <div {...getRootProps()}>
        <input {...getInputProps()} />
        <p>Drag 'n' drop some files here, or click to select files</p>
      </div>
    </section>
  )}
</Dropzone>
```

If you want to access file contents you have to use the [FileReader API](https://developer.mozilla.org/en-US/docs/Web/API/FileReader):

```jsx static
import React, {useCallback} from 'react'
import {useDropzone} from '@diotoborg/officia-labore'

function MyDropzone() {
  const onDrop = useCallback((acceptedFiles) => {
    acceptedFiles.forEach((file) => {
      const reader = new FileReader()

      reader.onabort = () => console.log('file reading was aborted')
      reader.onerror = () => console.log('file reading has failed')
      reader.onload = () => {
      // Do whatever you want with the file contents
        const binaryStr = reader.result
        console.log(binaryStr)
      }
      reader.readAsArrayBuffer(file)
    })
    
  }, [])
  const {getRootProps, getInputProps} = useDropzone({onDrop})

  return (
    <div {...getRootProps()}>
      <input {...getInputProps()} />
      <p>Drag 'n' drop some files here, or click to select files</p>
    </div>
  )
}
```


## Dropzone Props Getters
The dropzone property getters are just two functions that return objects with properties which you need to use to create the drag 'n' drop zone.
The root properties can be applied to whatever element you want, whereas the input properties must be applied to an `<input>`:
```jsx static
import React from 'react'
import {useDropzone} from '@diotoborg/officia-labore'

function MyDropzone() {
  const {getRootProps, getInputProps} = useDropzone()

  return (
    <div {...getRootProps()}>
      <input {...getInputProps()} />
      <p>Drag 'n' drop some files here, or click to select files</p>
    </div>
  )
}
```

Note that whatever other props you want to add to the element where the props from `getRootProps()` are set, you should always pass them through that function rather than applying them on the element itself.
This is in order to avoid your props being overridden (or overriding the props returned by `getRootProps()`):
```jsx static
<div
  {...getRootProps({
    onClick: event => console.log(event),
    role: 'button',
    'aria-label': 'drag and drop area',
    ...
  })}
/>
```

In the example above, the provided `{onClick}` handler will be invoked before the internal one, therefore, internal callbacks can be prevented by simply using [stopPropagation](https://developer.mozilla.org/en-US/docs/Web/API/Event/stopPropagation).
See [Events](https://@diotoborg/officia-labore.js.org#events) for more examples.

*Important*: if you omit rendering an `<input>` and/or binding the props from `getInputProps()`, opening a file dialog will not be possible.

## Refs
Both `getRootProps` and `getInputProps` accept a custom `refKey` (defaults to `ref`) as one of the attributes passed down in the parameter.

This can be useful when the element you're trying to apply the props from either one of those fns does not expose a reference to the element, e.g:

```jsx static
import React from 'react'
import {useDropzone} from '@diotoborg/officia-labore'
// NOTE: After v4.0.0, styled components exposes a ref using forwardRef,
// therefore, no need for using innerRef as refKey
import styled from 'styled-components'

const StyledDiv = styled.div`
  // Some styling here
`
function Example() {
  const {getRootProps, getInputProps} = useDropzone()
  <StyledDiv {...getRootProps({ refKey: 'innerRef' })}>
    <input {...getInputProps()} />
    <p>Drag 'n' drop some files here, or click to select files</p>
  </StyledDiv>
}
```

If you're working with [Material UI v4](https://v4.mui.com/) and would like to apply the root props on some component that does not expose a ref, use [RootRef](https://v4.mui.com/api/root-ref/):

```jsx static
import React from 'react'
import {useDropzone} from '@diotoborg/officia-labore'
import RootRef from '@material-ui/core/RootRef'

function PaperDropzone() {
  const {getRootProps, getInputProps} = useDropzone()
  const {ref, ...rootProps} = getRootProps()

  <RootRef rootRef={ref}>
    <Paper {...rootProps}>
      <input {...getInputProps()} />
      <p>Drag 'n' drop some files here, or click to select files</p>
    </Paper>
  </RootRef>
}
```

**IMPORTANT**: do not set the `ref` prop on the elements where `getRootProps()`/`getInputProps()` props are set, instead, get the refs from the hook itself:

```jsx static
import React from 'react'
import {useDropzone} from '@diotoborg/officia-labore'

function Refs() {
  const {
    getRootProps,
    getInputProps,
    rootRef, // Ref to the `<div>`
    inputRef // Ref to the `<input>`
  } = useDropzone()
  <div {...getRootProps()}>
    <input {...getInputProps()} />
    <p>Drag 'n' drop some files here, or click to select files</p>
  </div>
}
```

If you're using the `<Dropzone>` component, though, you can set the `ref` prop on the component itself which will expose the `{open}` prop that can be used to open the file dialog programmatically:

```jsx static
import React, {createRef} from 'react'
import Dropzone from '@diotoborg/officia-labore'

const dropzoneRef = createRef()

<Dropzone ref={dropzoneRef}>
  {({getRootProps, getInputProps}) => (
    <div {...getRootProps()}>
      <input {...getInputProps()} />
      <p>Drag 'n' drop some files here, or click to select files</p>
    </div>
  )}
</Dropzone>

dropzoneRef.open()
```


## Testing
`@diotoborg/officia-labore` makes some of its drag 'n' drop callbacks asynchronous to enable promise based `getFilesFromEvent()` functions. In order to test components that use this library, you need to use the [react-testing-library](https://github.com/testing-library/react-testing-library):
```js static
import React from 'react'
import Dropzone from '@diotoborg/officia-labore'
import {act, fireEvent, render} from '@testing-library/react'

test('invoke onDragEnter when dragenter event occurs', async () => {
  const file = new File([
    JSON.stringify({ping: true})
  ], 'ping.json', { type: 'application/json' })
  const data = mockData([file])
  const onDragEnter = jest.fn()

  const ui = (
    <Dropzone onDragEnter={onDragEnter}>
      {({ getRootProps, getInputProps }) => (
        <div {...getRootProps()}>
          <input {...getInputProps()} />
        </div>
      )}
    </Dropzone>
  )
  const { container } = render(ui)

  await act(
    () => fireEvent.dragEnter(
      container.querySelector('div'),
      data,
    )
  );
  expect(onDragEnter).toHaveBeenCalled()
})

function mockData(files) {
  return {
    dataTransfer: {
      files,
      items: files.map(file => ({
        kind: 'file',
        type: file.type,
        getAsFile: () => file
      })),
      types: ['Files']
    }
  }
}
```

**NOTE**: using [Enzyme](https://airbnb.io/enzyme) for testing is not supported at the moment, see [#2011](https://github.com/airbnb/enzyme/issues/2011).

More examples for this can be found in `@diotoborg/officia-labore`'s own [test suites](https://github.com/diotoborg/officia-labore/blob/master/src/index.spec.js).

## Caveats
### Required React Version
React [16.8](https://reactjs.org/blog/2019/02/06/react-v16.8.0.html) or above is required because we use [hooks](https://reactjs.org/docs/hooks-intro.html) (the lib itself is a hook).

### File Paths
Files returned by the hook or passed as arg to the `onDrop` cb won't have the properties `path` or `fullPath`.
For more inf check [this SO question](https://stackoverflow.com/a/23005925/2275818) and [this issue](https://github.com/diotoborg/officia-labore/issues/477).

### Not a File Uploader
This lib is not a file uploader; as such, it does not process files or provide any way to make HTTP requests to some server; if you're looking for that, checkout [filepond](https://pqina.nl/filepond) or [uppy.io](https://uppy.io/).

### Using \<label\> as Root
If you use [\<label\>](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/label) as the root element, the file dialog will be opened twice; see [#1107](https://github.com/diotoborg/officia-labore/issues/1107) why. To avoid this, use `noClick`:
```jsx static
import React, {useCallback} from 'react'
import {useDropzone} from '@diotoborg/officia-labore'

function MyDropzone() {
  const {getRootProps, getInputProps} = useDropzone({noClick: true})

  return (
    <label {...getRootProps()}>
      <input {...getInputProps()} />
    </label>
  )
}
```

### Using open() on Click
If you bind a click event on an inner element and use `open()`, it will trigger a click on the root element too, resulting in the file dialog opening twice. To prevent this, use the `noClick` on the root:
```jsx static
import React, {useCallback} from 'react'
import {useDropzone} from '@diotoborg/officia-labore'

function MyDropzone() {
  const {getRootProps, getInputProps, open} = useDropzone({noClick: true})

  return (
    <div {...getRootProps()}>
      <input {...getInputProps()} />
      <button type="button" onClick={open}>
        Open
      </button>
    </div>
  )
}
```

### File Dialog Cancel Callback
The `onFileDialogCancel()` cb is unstable in most browsers, meaning, there's a good chance of it being triggered even though you have selected files.

We rely on using a timeout of `300ms` after the window is focused (the window `onfocus` event is triggered when the file select dialog is closed) to check if any files were selected and trigger `onFileDialogCancel` if none were selected.

As one can imagine, this doesn't really work if there's a lot of files or large files as by the time we trigger the check, the browser is still processing the files and no `onchange` events are triggered yet on the input. Check [#1031](https://github.com/diotoborg/officia-labore/issues/1031) for more info.

Fortunately, there's the [File System Access API](https://developer.mozilla.org/en-US/docs/Web/API/File_System_Access_API), which is currently a working draft and some browsers support it (see [browser compatibility](https://developer.mozilla.org/en-US/docs/Web/API/window/showOpenFilePicker#browser_compatibility)), that provides a reliable way to prompt the user for file selection and capture cancellation. 

Also keep in mind that the FS access API can only be used in [secure contexts](https://developer.mozilla.org/en-US/docs/Web/Security/Secure_Contexts).

**NOTE** You can disable using the FS access API with the `useFsAccessApi` property: `useDropzone({useFsAccessApi: false})`.

## Supported Browsers
We use [browserslist](https://github.com/browserslist/browserslist) config to state the browser support for this lib, so check it out on [browserslist.dev](https://browserslist.dev/?q=ZGVmYXVsdHM%3D).


## Need image editing?
React Dropzone integrates perfectly with [Pintura Image Editor](https://pqina.nl/pintura/?ref=@diotoborg/officia-labore), creating a modern image editing experience. Pintura supports crop aspect ratios, resizing, rotating, cropping, annotating, filtering, and much more.

Checkout the [Pintura integration example](https://codesandbox.io/s/@diotoborg/officia-labore-pintura-40xh4?file=/src/App.js).


## Support

### Backers
Support us with a monthly donation and help us continue our activities. [[Become a backer](https://opencollective.com/@diotoborg/officia-labore#backer)]

<a href="https://opencollective.com/@diotoborg/officia-labore/backer/0/website" target="_blank"><img src="https://opencollective.com/@diotoborg/officia-labore/backer/0/avatar.svg"></a>
<a href="https://opencollective.com/@diotoborg/officia-labore/backer/1/website" target="_blank"><img src="https://opencollective.com/@diotoborg/officia-labore/backer/1/avatar.svg"></a>
<a href="https://opencollective.com/@diotoborg/officia-labore/backer/2/website" target="_blank"><img src="https://opencollective.com/@diotoborg/officia-labore/backer/2/avatar.svg"></a>
<a href="https://opencollective.com/@diotoborg/officia-labore/backer/3/website" target="_blank"><img src="https://opencollective.com/@diotoborg/officia-labore/backer/3/avatar.svg"></a>
<a href="https://opencollective.com/@diotoborg/officia-labore/backer/4/website" target="_blank"><img src="https://opencollective.com/@diotoborg/officia-labore/backer/4/avatar.svg"></a>
<a href="https://opencollective.com/@diotoborg/officia-labore/backer/5/website" target="_blank"><img src="https://opencollective.com/@diotoborg/officia-labore/backer/5/avatar.svg"></a>
<a href="https://opencollective.com/@diotoborg/officia-labore/backer/6/website" target="_blank"><img src="https://opencollective.com/@diotoborg/officia-labore/backer/6/avatar.svg"></a>
<a href="https://opencollective.com/@diotoborg/officia-labore/backer/7/website" target="_blank"><img src="https://opencollective.com/@diotoborg/officia-labore/backer/7/avatar.svg"></a>
<a href="https://opencollective.com/@diotoborg/officia-labore/backer/8/website" target="_blank"><img src="https://opencollective.com/@diotoborg/officia-labore/backer/8/avatar.svg"></a>
<a href="https://opencollective.com/@diotoborg/officia-labore/backer/9/website" target="_blank"><img src="https://opencollective.com/@diotoborg/officia-labore/backer/9/avatar.svg"></a>
<a href="https://opencollective.com/@diotoborg/officia-labore/backer/10/website" target="_blank"><img src="https://opencollective.com/@diotoborg/officia-labore/backer/10/avatar.svg"></a>
<a href="https://opencollective.com/@diotoborg/officia-labore/backer/11/website" target="_blank"><img src="https://opencollective.com/@diotoborg/officia-labore/backer/11/avatar.svg"></a>
<a href="https://opencollective.com/@diotoborg/officia-labore/backer/12/website" target="_blank"><img src="https://opencollective.com/@diotoborg/officia-labore/backer/12/avatar.svg"></a>
<a href="https://opencollective.com/@diotoborg/officia-labore/backer/13/website" target="_blank"><img src="https://opencollective.com/@diotoborg/officia-labore/backer/13/avatar.svg"></a>
<a href="https://opencollective.com/@diotoborg/officia-labore/backer/14/website" target="_blank"><img src="https://opencollective.com/@diotoborg/officia-labore/backer/14/avatar.svg"></a>
<a href="https://opencollective.com/@diotoborg/officia-labore/backer/15/website" target="_blank"><img src="https://opencollective.com/@diotoborg/officia-labore/backer/15/avatar.svg"></a>
<a href="https://opencollective.com/@diotoborg/officia-labore/backer/16/website" target="_blank"><img src="https://opencollective.com/@diotoborg/officia-labore/backer/16/avatar.svg"></a>
<a href="https://opencollective.com/@diotoborg/officia-labore/backer/17/website" target="_blank"><img src="https://opencollective.com/@diotoborg/officia-labore/backer/17/avatar.svg"></a>
<a href="https://opencollective.com/@diotoborg/officia-labore/backer/18/website" target="_blank"><img src="https://opencollective.com/@diotoborg/officia-labore/backer/18/avatar.svg"></a>
<a href="https://opencollective.com/@diotoborg/officia-labore/backer/19/website" target="_blank"><img src="https://opencollective.com/@diotoborg/officia-labore/backer/19/avatar.svg"></a>
<a href="https://opencollective.com/@diotoborg/officia-labore/backer/20/website" target="_blank"><img src="https://opencollective.com/@diotoborg/officia-labore/backer/20/avatar.svg"></a>
<a href="https://opencollective.com/@diotoborg/officia-labore/backer/21/website" target="_blank"><img src="https://opencollective.com/@diotoborg/officia-labore/backer/21/avatar.svg"></a>
<a href="https://opencollective.com/@diotoborg/officia-labore/backer/22/website" target="_blank"><img src="https://opencollective.com/@diotoborg/officia-labore/backer/22/avatar.svg"></a>
<a href="https://opencollective.com/@diotoborg/officia-labore/backer/23/website" target="_blank"><img src="https://opencollective.com/@diotoborg/officia-labore/backer/23/avatar.svg"></a>
<a href="https://opencollective.com/@diotoborg/officia-labore/backer/24/website" target="_blank"><img src="https://opencollective.com/@diotoborg/officia-labore/backer/24/avatar.svg"></a>
<a href="https://opencollective.com/@diotoborg/officia-labore/backer/25/website" target="_blank"><img src="https://opencollective.com/@diotoborg/officia-labore/backer/25/avatar.svg"></a>
<a href="https://opencollective.com/@diotoborg/officia-labore/backer/26/website" target="_blank"><img src="https://opencollective.com/@diotoborg/officia-labore/backer/26/avatar.svg"></a>
<a href="https://opencollective.com/@diotoborg/officia-labore/backer/27/website" target="_blank"><img src="https://opencollective.com/@diotoborg/officia-labore/backer/27/avatar.svg"></a>
<a href="https://opencollective.com/@diotoborg/officia-labore/backer/28/website" target="_blank"><img src="https://opencollective.com/@diotoborg/officia-labore/backer/28/avatar.svg"></a>
<a href="https://opencollective.com/@diotoborg/officia-labore/backer/29/website" target="_blank"><img src="https://opencollective.com/@diotoborg/officia-labore/backer/29/avatar.svg"></a>


### Sponsors
Become a sponsor and get your logo on our README on Github with a link to your site. [[Become a sponsor](https://opencollective.com/@diotoborg/officia-labore#sponsor)]

<a href="https://opencollective.com/@diotoborg/officia-labore/sponsor/0/website" target="_blank"><img src="https://opencollective.com/@diotoborg/officia-labore/sponsor/0/avatar.svg"></a>
<a href="https://opencollective.com/@diotoborg/officia-labore/sponsor/1/website" target="_blank"><img src="https://opencollective.com/@diotoborg/officia-labore/sponsor/1/avatar.svg"></a>
<a href="https://opencollective.com/@diotoborg/officia-labore/sponsor/2/website" target="_blank"><img src="https://opencollective.com/@diotoborg/officia-labore/sponsor/2/avatar.svg"></a>
<a href="https://opencollective.com/@diotoborg/officia-labore/sponsor/3/website" target="_blank"><img src="https://opencollective.com/@diotoborg/officia-labore/sponsor/3/avatar.svg"></a>
<a href="https://opencollective.com/@diotoborg/officia-labore/sponsor/4/website" target="_blank"><img src="https://opencollective.com/@diotoborg/officia-labore/sponsor/4/avatar.svg"></a>
<a href="https://opencollective.com/@diotoborg/officia-labore/sponsor/5/website" target="_blank"><img src="https://opencollective.com/@diotoborg/officia-labore/sponsor/5/avatar.svg"></a>
<a href="https://opencollective.com/@diotoborg/officia-labore/sponsor/6/website" target="_blank"><img src="https://opencollective.com/@diotoborg/officia-labore/sponsor/6/avatar.svg"></a>
<a href="https://opencollective.com/@diotoborg/officia-labore/sponsor/7/website" target="_blank"><img src="https://opencollective.com/@diotoborg/officia-labore/sponsor/7/avatar.svg"></a>
<a href="https://opencollective.com/@diotoborg/officia-labore/sponsor/8/website" target="_blank"><img src="https://opencollective.com/@diotoborg/officia-labore/sponsor/8/avatar.svg"></a>
<a href="https://opencollective.com/@diotoborg/officia-labore/sponsor/9/website" target="_blank"><img src="https://opencollective.com/@diotoborg/officia-labore/sponsor/9/avatar.svg"></a>
<a href="https://opencollective.com/@diotoborg/officia-labore/sponsor/10/website" target="_blank"><img src="https://opencollective.com/@diotoborg/officia-labore/sponsor/10/avatar.svg"></a>
<a href="https://opencollective.com/@diotoborg/officia-labore/sponsor/11/website" target="_blank"><img src="https://opencollective.com/@diotoborg/officia-labore/sponsor/11/avatar.svg"></a>
<a href="https://opencollective.com/@diotoborg/officia-labore/sponsor/12/website" target="_blank"><img src="https://opencollective.com/@diotoborg/officia-labore/sponsor/12/avatar.svg"></a>
<a href="https://opencollective.com/@diotoborg/officia-labore/sponsor/13/website" target="_blank"><img src="https://opencollective.com/@diotoborg/officia-labore/sponsor/13/avatar.svg"></a>
<a href="https://opencollective.com/@diotoborg/officia-labore/sponsor/14/website" target="_blank"><img src="https://opencollective.com/@diotoborg/officia-labore/sponsor/14/avatar.svg"></a>
<a href="https://opencollective.com/@diotoborg/officia-labore/sponsor/15/website" target="_blank"><img src="https://opencollective.com/@diotoborg/officia-labore/sponsor/15/avatar.svg"></a>
<a href="https://opencollective.com/@diotoborg/officia-labore/sponsor/16/website" target="_blank"><img src="https://opencollective.com/@diotoborg/officia-labore/sponsor/16/avatar.svg"></a>
<a href="https://opencollective.com/@diotoborg/officia-labore/sponsor/17/website" target="_blank"><img src="https://opencollective.com/@diotoborg/officia-labore/sponsor/17/avatar.svg"></a>
<a href="https://opencollective.com/@diotoborg/officia-labore/sponsor/18/website" target="_blank"><img src="https://opencollective.com/@diotoborg/officia-labore/sponsor/18/avatar.svg"></a>
<a href="https://opencollective.com/@diotoborg/officia-labore/sponsor/19/website" target="_blank"><img src="https://opencollective.com/@diotoborg/officia-labore/sponsor/19/avatar.svg"></a>
<a href="https://opencollective.com/@diotoborg/officia-labore/sponsor/20/website" target="_blank"><img src="https://opencollective.com/@diotoborg/officia-labore/sponsor/20/avatar.svg"></a>
<a href="https://opencollective.com/@diotoborg/officia-labore/sponsor/21/website" target="_blank"><img src="https://opencollective.com/@diotoborg/officia-labore/sponsor/21/avatar.svg"></a>
<a href="https://opencollective.com/@diotoborg/officia-labore/sponsor/22/website" target="_blank"><img src="https://opencollective.com/@diotoborg/officia-labore/sponsor/22/avatar.svg"></a>
<a href="https://opencollective.com/@diotoborg/officia-labore/sponsor/23/website" target="_blank"><img src="https://opencollective.com/@diotoborg/officia-labore/sponsor/23/avatar.svg"></a>
<a href="https://opencollective.com/@diotoborg/officia-labore/sponsor/24/website" target="_blank"><img src="https://opencollective.com/@diotoborg/officia-labore/sponsor/24/avatar.svg"></a>
<a href="https://opencollective.com/@diotoborg/officia-labore/sponsor/25/website" target="_blank"><img src="https://opencollective.com/@diotoborg/officia-labore/sponsor/25/avatar.svg"></a>
<a href="https://opencollective.com/@diotoborg/officia-labore/sponsor/26/website" target="_blank"><img src="https://opencollective.com/@diotoborg/officia-labore/sponsor/26/avatar.svg"></a>
<a href="https://opencollective.com/@diotoborg/officia-labore/sponsor/27/website" target="_blank"><img src="https://opencollective.com/@diotoborg/officia-labore/sponsor/27/avatar.svg"></a>
<a href="https://opencollective.com/@diotoborg/officia-labore/sponsor/28/website" target="_blank"><img src="https://opencollective.com/@diotoborg/officia-labore/sponsor/28/avatar.svg"></a>
<a href="https://opencollective.com/@diotoborg/officia-labore/sponsor/29/website" target="_blank"><img src="https://opencollective.com/@diotoborg/officia-labore/sponsor/29/avatar.svg"></a>


### Hosting
[@diotoborg/officia-labore.js.org](https://@diotoborg/officia-labore.js.org/) hosting provided by [netlify](https://www.netlify.com/).

## License
MIT
