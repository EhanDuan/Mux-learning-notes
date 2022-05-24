# Context
## Demand
The company needs to realize a page for users to upload their videos online. In the face, the user flow is:
+ click `select` button
+ choose a video
+ upload the video online
<img width="921" alt="image" src="https://user-images.githubusercontent.com/75397936/170139264-d3a838fe-6f24-42f6-a60c-85f39feb0eff.png">

## Environment
Stream Video API we are using is `Mux`. So videos should upload into `Mux`. 
For general data storage, firestore is chosen. Here, `video` data should be stored in the `video` collections. For each video, there will be a document for record.
In addition, videos cannot store in the web server. In future, the videos will be uploaded in some long-term storage database.
<img width="922" alt="image" src="https://user-images.githubusercontent.com/75397936/170136715-8ef52907-f8ec-401a-8894-0106d3ba0d30.png">

## Logic
<img width="989" alt="image" src="https://user-images.githubusercontent.com/75397936/170136852-fb33af29-57aa-44f3-aedb-9ad8f098ff71.png">

Transaction is just my thought and doubt: how to implement transcation to group some operations together? Still no answer.


# Code From Mux Platform 
Here is the implementations from NextJs-Mux and I write some comments based on Mux documentations to help me better understanding the big picture.
Project Demo: https://with-mux-video.vercel.app.
Project Code: https://github.com/vercel/next.js/tree/canary/examples/with-mux-video.

**Others Useful:**

`UpChunk` source code: https://github.com/muxinc/upchunk.

## Steps<br>
Reference: [Upload files directly](https://docs.mux.com/guides/video/upload-files-directly#creating-an-upload-route-in-the-application). <br>
**1. Create an authenticated Mux URL** <br>

This file handles the request to upload a selected video and return {id: upload.id, url: upload.url}. `upload.url` is the url for client to upload.<br>

 `api/upload.js`<br>

```js
import Mux from '@mux/mux-node'
const { Video } = new Mux()

export default async function uploadHandler(req, res) {
  const { method } = req

  switch (method) {
    case 'POST':
      try {
        const upload = await Video.Uploads.create({
          new_asset_settings: { playback_policy: 'public' },
          cors_origin: '*',
        })
        res.json({
          id: upload.id,
          url: upload.url,
        })
      } catch (e) {
        console.error('Request error', e)
        res.status(500).json({ error: 'Error creating upload' })
      }
      break
    default:
      res.setHeader('Allow', ['POST'])
      res.status(405).end(`Method ${method} Not Allowed`)
  }
}

```
**2. Use that URL to upload in your client** <br>
Part of File : `Components/upload-form.js` <br>
```js
const startUpload = (evt) => {
    setIsUploading(true)
    const upload = UpChunk.createUpload({
      endpoint: createUpload,
      file: inputRef.current.files[0],
    })

    upload.on('error', (err) => {
      setErrorMessage(err.detail)
    })

    upload.on('progress', (progress) => {
      setProgress(Math.floor(progress.detail))
    })

    upload.on('success', () => {
      setIsPreparing(true)
    })
  }
```

`Components/upload-form.js`
```js
import { useEffect, useRef, useState } from 'react'
import Router from 'next/router'
import * as UpChunk from '@mux/upchunk'
import useSwr from 'swr'
import Button from './button'
import Spinner from './spinner'
import ErrorMessage from './error-message'

const fetcher = (url) => {
  return fetch(url).then((res) => res.json())
}

const UploadForm = () => {
  const [isUploading, setIsUploading] = useState(false)
  const [isPreparing, setIsPreparing] = useState(false)
  const [uploadId, setUploadId] = useState(null)
  const [progress, setProgress] = useState(null)
  const [errorMessage, setErrorMessage] = useState('')
  const inputRef = useRef(null)

  const { data, error } = useSwr(
    () => (isPreparing ? `/api/upload/${uploadId}` : null),
    fetcher,
    { refreshInterval: 5000 }
  )

  const upload = data && data.upload

  useEffect(() => {
    if (upload && upload.asset_id) {
      Router.push({
        pathname: `/asset/${upload.asset_id}`,
        scroll: false,
      })
    }
  }, [upload])

  if (error) return <ErrorMessage message="Error fetching api" />
  if (data && data.error) return <ErrorMessage message={data.error} />

  const createUpload = async () => {
    try {
      return fetch('/api/upload', {
        method: 'POST',
      })
        .then((res) => res.json())
        .then(({ id, url }) => {
          setUploadId(id)
          return url
        })
    } catch (e) {
      console.error('Error in createUpload', e)
      setErrorMessage('Error creating upload')
    }
  }

  const startUpload = (evt) => {
    setIsUploading(true)
    const upload = UpChunk.createUpload({
      endpoint: createUpload,
      file: inputRef.current.files[0],
    })

    upload.on('error', (err) => {
      setErrorMessage(err.detail)
    })

    upload.on('progress', (progress) => {
      setProgress(Math.floor(progress.detail))
    })

    upload.on('success', () => {
      setIsPreparing(true)
    })
  }

  if (errorMessage) return <ErrorMessage message={errorMessage} />

  return (
    <>
      <div className="container">
        {isUploading ? (
          <>
            {isPreparing ? (
              <div>Preparing..</div>
            ) : (
              <div>Uploading...{progress ? `${progress}%` : ''}</div>
            )}
            <Spinner />
          </>
        ) : (
          <label>
            <Button type="button" onClick={() => inputRef.current.click()}>
              Select a video file
            </Button>
            <input type="file" onChange={startUpload} ref={inputRef} />
          </label>
        )}
      </div>
      <style jsx>{`
        input {
          display: none;
        }
      `}</style>
    </>
  )
}

export default UploadForm

```
