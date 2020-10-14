# Let Get Started
  As a user of your mobile or web app, I want to personalize my experience by including profile picture 😎 but the picture I want to use is too large and does not focus the part I wanted or I want to remove some part of the image. 
It will be nice to use the app to crop the image to my taste! Right?

  Apart from the editting part, for better image experience, we might need images of a particular width and height ratio (📝 we call this **aspect ratio**) in our apps. 
In this article, we are going to implement a simple image cropping system in a React Application we will create. Check out the result demo [here](https://Joshua0707.github.io/Image-Crop-In-React) . Great! Let's get started! 🚀.
 
# Prerequisites
  Yeah! Knowledge of Javascript, CSS and React is required but if your are a beginner or intermediate with React and Javascript, why not join the train 🚆.

# Project Structure
  Let's create our React projects with our command line.
```bash
npm init react-app reactimagecrop
```
After our **reactimagecrop** project is loaded, we are left with a nice structure. We won't need some files like `logo.svg` and the test files. I included `components` folder (which will hold the components we will use), `helper` folder for utilities files.  So, I am left
* public
* src
  * components
  * helpers
  * app.js
  * app.css
  * index.js
  * index.css
  * serviceWorker.js

We will also use a react image cropping component, **ReactCrop**. You could check out it's documentation here.
```bash
npm install ReactCrop
```
Take a peek at my package.json
```json
{
  "name": "image-cropping-in-react"
  
  "dependencies": {
    "@testing-library/jest-dom": "^4.2.4",
    "@testing-library/react": "^9.5.0",
    "@testing-library/user-event": "^7.2.1",
    "gh-pages": "^3.1.0",
    "react": "^16.13.1",
    "react-dom": "^16.13.1",
    "react-image-crop": "^8.6.6",
    "react-scripts": "3.4.3"
  },
  
}
```

This tutorial will be segmented based on these tasks or should we call it todo?
* Creating and Styling the Upload Page
* Creating the Crop Modal Component
* Using the Crop Modal Component
* Making Image Preview from Upload Inputs
* Using the ReactImageCrop component
* Adding Custom Styles to ReactImageCrop
* Creating the Result Container
* Cropping the Image with Canvas
* Validating Upload Inputs

# Creating and Styling the Upload Page
We are creating this page in our `app.js` file. We have the react template code here, we don't need some elements and so we remove them. We are left with this.
```jsx
import React from 'react';
import './App.css';

function App() {
  return (
    <div className="App">

    </div>
  )
}

export default App;
```
So we include `input[type="file"]` tag for uploading files into the `div.upload-bx` container. We need this for our custom styling 🌈 for the upload.
```jsx
// ...
return (
  <div className="App">
    <div className="upload-bx">
      <input type="file" onInput={handleInput} accept="image/gif, image/jpeg, image/png, image/jpg" />
      <span>{ handUp }</span>
      <span>Upload</span>
    </div>
  </div>
)
...
```

I included a svg import from my project file. If you need this file check it in [here](https://Joshua0707.github.io/Image-Crop-In-React). Now we style, in `app.css`. This is the trick! Since we're making our custom input file upload element, we use positioning to make the `input` fill the box and we reduce its opacity to 0. Then, the `input` is there but we can't see it.
```css
.App .upload-bx {
  width: 400px;
  height: 400px;
  /* to support 'absolute'ly positioned children */
  position: relative;
}

.App .upload-bx [type=file] {
  /* fill its parent */
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  /* make invisible */
  opacity: 0;
  cursor: pointer;
}
```

You can style it to your taste 👌. 

# Creating the Crop Modal Component
We are making the crop component a modal. In the `components` folder, I have the `ImageCropModal.js` file. That's the crop modal component. For simplicity, we are making use of *React's class component*.
```jsx
class ImageCropModal extends React.Component {
  render() {
    return (
      <main className="image-modal">
        <div className="image-crop-bx">
          <div className="modal-toolbar">
              <span>{ times }</span>
              <span>{ check }</span>
          </div>
          <div className="modal-crop-bx"></div>
        </div>
      </main>
    )
  }
}

export default ImageCropModal;
```
Wow! We have a nest of containers. The `main.image-modal` fills the entire screen with a dark translucent background. `div.image-crop-bx` is the modal content, the container that houses most importantly the crop component. We have the `div.modal-toolbar` that houses icons for closing the modal, cancelling selection and the other for choosing crop selection. The `div.modal-crop-bx` houses the `ReactImageCrop` component we installed already via npm.

In the same folder `components` folder, I have `imagecropmodal.css` for styling. We import it in the `ImageCropModal.js` as the following.
```jsx
...
import './imagecropmodal.css';
...
```

Now, let's work on the styling for the modal component.
```css
.image-modal {
  /* Fills the screen */
  width: 100vw;
  height: 100vh;
  position: fixed;
  top: 0;
  z-index: 100;
  /* Set the dark translucent background */
  background-color: rgba(0, 0, 0, 0.555);
  /* Position direct children (.image-crop-bx) to center */
  display: grid;
  place-items: center;
}

.image-modal .image-crop-bx {
  width: 600px;
  height: 600px;
  /* align children in a column */
  display: flex;
  flex-flow: column nowrap;
  /* my styling */
  border-radius: 8px;
  background-color: white;
  padding: 1rem;
  box-sizing: border-box;
}

.image-crop-bx .modal-toolbar {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 20px;
    height: 30px;
    box-sizing: border-box;
}

.image-crop-bx .modal-toolbar span {
    background-color: black;
    width: 30px;
    height: 30px;
    border-radius: 50%;
    display: grid;
    place-items: center;
    cursor: pointer;
}

.image-crop-bx .modal-crop-bx {
    margin: auto;
    width: 100%;
    height: calc(100% - 50px);
    background-color: #c0c0c0;
}
```

You could do some other works on the styling regarding the toolbar's button colors and other criteria.

# Using the Upload Modal Component
In the last section, we just created the Crop Modal Component. Time ⌛ to implement it. We want the crop modal to come up when we have selected an image. Let's use this logic. We have a `image-input` state variable initialized to `null`. When `image-input` is `null`, the modal is not rendered else the modal is rendered. In `App.js`, we could do this
```jsx
// ...
function App() {
  const [ imageInput, setImageInput ] = React.useState(null); // initialised to null

  return (
    <div className="App">
    {
      imageInput ? ( {/* imageInput is not null */}
        <div className="upload-bx">
          <input type="file" />
          <span>{ handUp }</span>
          <span>Upload</span>
        </div>
      ) : ( {/* else */}
        <ImageCropModal></ImageCropModal>
      )
    } 
  );
}
```

Let's update this to be the selected file via our upload component. I created a `handleInput` function to do just that. The `onInput` event triggered as a selection is made in the input file gives an *event* parameter. It comprises of all information about the event occurred. Part of that information is the `target`, which is the element (, in this case the `input[type="file"]` element) the event occurred on. We could get the value of the input but for file type, we get the files selected by get its `files` attribute. We are in this tutorial selecting just one, the first one.
```jsx
// ...
function App() {
  // ...
  const handleInput = (e) => {
    setImageInput(e.target.files[0]) //  getting the first selected file if multiple file is selected
  }

  return (
    <div className="App">
    {/* ... */}
        <div className="upload-bx">
          <input type="file" onInput={handleInput} />
          <span>{ handUp }</span>
          <span>Upload</span>
        </div>
    {/* ... */}
    </div>
  );
}
```

# Making Image Preview from Upload Inputs
Until now, we just have the modal coming up when a file is selected and nothing else happens. Have you wondered how are files selected by input are displayed? especially without having uploaded it? Have you tried this before? Let me introduce you to what is called `base64`. `base64` is a binary-to-text encoding scheme. Read more on [here]("kds"). It can be included in a *data URL*. That's what we need to preview the selected image.

We should be able to pass the selected file to the `ImageCropModal` component as props. Just after render, we should be able to display the preview.
```jsx
class ImageCropModal extends React.Component {
  constructor(props) {
      super(props);
      this.state = {
        imgSrc: null,
      }
  }
  componentDidMount() { // immediately after render
    // extract the file
    const { imageSrc } = this.props;
    const reader = new FileReader();
    reader.addEventListener("load", () => {
      // set the imgSrc state variable the result dataURL
      this.setState({ imgSrc: reader.result })
    }, false);
    // create a base64 image file
    reader.readAsDataURL(imageSrc);
  }

  render() {
    return (
      <main className="image-modal">
        {/* ... */}
        <div className="modal-crop-bx">
            {  ? <img src={this.state.imgSrc} alt="preview" /> : <></> }
        </div>
          {/* ... */}
      </main>
    )
  }
}
```

This is not what we are developing but what I'm driving at is how to preview images from files. This is what the ReactImageCrop components needs to display the images selected.

# Using the ReactImageCrop component
Great Work! 💪 This is exactly have of our tasks. Instead of the `img` tag for preview, we want the `ReactImageCrop` component for cropping 🥂. 
```jsx
// ...
import ReactImageCrop from 'react-image-crop';

class ImageCropModal extends React.Component {
  // ...
  render() {
    return (
      <main className="image-modal">
        {/* ... */}
        <div className="modal-crop-bx">
            <ReactImageCrop src={this.state.imgSrc} />
        </div>
          {/* ... */}
      </main>
    )
  }
}
```

`ReactImageCrop` takes more than `src`. Check its documentation [here](link).  