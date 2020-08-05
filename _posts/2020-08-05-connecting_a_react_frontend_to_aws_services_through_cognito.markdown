---
layout: post
title:      "Connecting a React Frontend to AWS services through Cognito"
date:       2020-08-05 18:28:05 +0000
permalink:  connecting_a_react_frontend_to_aws_services_through_cognito
---


I followed this [tutorial](http://docs.aws.amazon.com/sdk-for-javascript/v2/developer-guide/getting-started-browser.html) to setup Amazon Polly and Cognito.  This configuration allows the frontend to interact directly with AWS services.  The tutorial is pretty straightforward, but I've documented a few deviations here.

### Installing AWS SDK
First of all, instead of including the AWS SDK with a script tag I [installed it with npm](http://aws.amazon.com/blogs/mobile/integrate-the-aws-sdk-for-javascript-into-a-react-app/
). `npm install aws-sdk`

In my React component

```
import AWS from 'aws-sdk'
```

### What to do with credentials

Instead of adding the region and identity pool id directly to the component I added them to a .env file.  According to [Create React App](http://create-react-app.dev/docs/adding-custom-environment-variables/) environement variables need to start with `REACT_APP_` and are accessed with `process.env.REACT_APP_VARIABLE_NAME`.  However Create React App docs also say that all .env files should be checked into source control. Saving this information in environment variables is really just for convenience and does not provide any kind of security.

This brings up an interesting question:  

### Is it secure to expose the region and identity pool id?

Accoding to [AWS](http://forums.aws.amazon.com/thread.jspa?threadID=245752&tstart=200) the Cognito identity pool id should only be used to allow login and signup related services.  This tutorial allows an unauthorized guest user to access Polly for the sake of simplicity in the example.  So it should be safe to expose this information with the proper user authorization in place.

### Adjusting the example with React hooks


The tutorial is in pure javascript so I adjusted it a bit for React. I added react hooks to update the dom for the input text, audio source and result notice.

```
const [text, setText] = useState('What would you like me to say?')
const [audioSource, setAudioSource] = useState('')
const [result, setResult] = useState("Enter text above then click Synthesize")
```

I made the input value controlled:

```
<input onChange={event => handleInputChange(event)} autoFocus size="23" type="text" id="textEntry" value={text}/>
```

```
const handleInputChange = (event) => {
        setText(event.target.value)
    }
```

### Controlling Audio load with React refs

The only part of the example code that still felt "un-React-ish" was quering the document for the audio element in order to call `.load()`.  According to the [React docs](http://reactjs.org/docs/refs-and-the-dom.html#when-to-use-refs), handling media playback is a good time to use refs.

Refs cannot be used on [functional components](http://reactjs.org/docs/refs-and-the-dom.html#refs-and-function-components) but they can be used inside a functional component.  Instead of using `React.createRef()` import `useRef` from React and creat a variable inside the functional component.

```
import React, {useState, useRef} from 'react'
```

Inside the component:

```
const audioRef = useRef(null)
```

Assign it to the audio element inside the return block:

```
<audio ref={audioRef} id="audioPlayback" controls>
      <source id="audioSource" type="audio/mp3" src={audioSource} />
</audio>
```

Access the element at audioRef.current:

```
audioRef.current.load()
```


