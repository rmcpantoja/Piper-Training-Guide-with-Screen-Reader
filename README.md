# Piper-Training-Guide-with-Screen-Reader

A guide to help newcomers to the Piper TTS system create voices for NVDA and other screen readers down the line.

# Introduction

Welcome to this guide on training your own custom TTS voices using [Piper](https://github.com/rhasspy/piper) a fast and local text to speech engine optimized for low end hardware such as the Raspberry Pi. Unlike a lot of TTS engines that blind people might be familiar with, piper is based on some of the latest advancements in machine learning for speech synthesis. Under the hood, it uses [VITS](https://github.com/jaywalnut310/vits), an end to end speech synthesis model, and exports voices to the [Onnx runtime](https://github.com/microsoft/onnxruntime). We will be using Piper with [NVDA](https://www.nvaccess.org/download/) a free and open source screen reader for the Windows operating system. Currently, this is the only screen reader that is supported, however as the synthesizer develops I'm sure more platforms will be added in the future.

With Piper being based on ML, a lot of people may understandably have concerns about its performance. As of right now, the model is what I would characterize as "acceptable" for a screen reader user. This will doubtless be improved as updates are made, but I personally think it's good enough to do basic web browsing, check email, read social media, etc. The AddOn we will be using also supports speeding the voices up quite considerably without audio degradation, see `Noise w` parametter.

# Getting Things Ready

Before we have fun training a model, let's get everything set up first so we can test right away. [Click here](https://github.com/mush42/piper-nvda/releases/download/v2.0-beta2/piper_neural_voices-2.0-beta.nvda-addon) to download the AddOn directly. If you would like to learn more or get updates, [click here.](https://github.com/mush42/piper-nvda?ref=building.open-home.io) after installation, a dialogue will pop up when you next restart NVDA, explaining that you do not currently have voices installed and offering to take you to the Piper sample page where you can preview and download any voices you would like. Installing a new voice is very easy, simply go to the NVDA menu / piper voice manager option. In the installed tab, you will then find a button to install voices from a local archive, and it's as simple as choosing the one you want and pressing enter.

Also, you can download [new voices](https://huggingface.co/rhasspy/piper-voices/tree/v1.0.0) hosted in `🤗huggingface`. Simply go to the `downloadable` tab, choose your preferred language and the list of voices will appear. This list can contain as many voices as are trained in that language. You can listen to a sample and download it, as well as refresh the list of voices (requires internet connection).

Take some time to play around with the different voices that are available and get a feel for how the synth operates, then come back and we'll get to the fun stuff.

# Creating Datasets

In any ML task, collecting sufficient data is probably one of the most important things you can do to get a high quality result. For TTS, this data should include audio files split at sentence breaks of a single speaker reading from a script, along with an accompanying text transcript. Piper makes your job easier by keeping the format very simple. For the audio, you can either use 16 or 22.050 kHz mono .wav files at 16 bit resolution, with durations between 8-15 seconds. For the text, you should format it according to the popular LJSpeech conventions, where the first column contains the name of your audio file with or without the extension, and the second column contains the text transcript, with each separated by a pipe character. The format looks like this:
```
audio1|This is the first sentence.
audio2| This is the second sentence.
```

Note that unlike LJSpeech, you do not need to repeat the text transcript a second time, as that is reserved for multi speaker models when providing speaker IDs. Note also that it is totally fine to have the path to your audio files before the file name, but it is not necessary, as Piper will handle all of that for you behind the scenes.

## Tips For Collecting Data

When working with TTS models, the quality of your data is very important. If your data has background noise, other people talking, etc, you will not get a satisfactory result, and the model will have a harder time learning the characteristics of your speaker. Also, if the voice contains background music, it is recommended to remove it with a vocal separator software like [mvsep](https://mvsep.com/). If possible, studio quality recordings are ideal for this kind of work, but even a laptop mic and a relatively low noise environment should do the trick as well. For scripts, try to find text that has wide phoneme coverage, such as public domain books or Wikipedia articles. I'll link to a few example Scripps at the end of this guide for you to get started. The amount of data to use is completely up to you, however I would recommend at least five minutes to begin. While that might sound like an insufficient amount to people familiar with older TTS systems, such as concatenation based synthesis, with machine learning it is quite different. You do not need nearly as much data to produce a high-quality result, and even just an hour of speech will get you a voice that sounds great in most scenarios. Obviously the more data the better, but don't feel like you have to get everything all at once. You can always go back and retrain the model later.

# Training Your Model

To train, we Will be using a service by Google called [colaboratory](https://colab.research.google.com/). This is a Jupiter notebook based environment that lets you get access to a high powered GPU in the cloud for free. Most TTS models based on ML require a high end machine with an Nvidia GPU for training, but can run on lower end hardware when synthesizing audio. Piper is no different, although it can also work on CPU when training, but it will be much slower. Here is a [link to a Colab notebook](https://colab.research.google.com/github/rmcpantoja/piper/blob/master/notebooks/piper_multilingual_training_notebook.ipynb) that will allow you to perform the model training. After training is complete, you must export your model to the Onnx runtime environment for use with the speech engine, and this [notebook](https://colab.research.google.com/github/rmcpantoja/piper/blob/master/notebooks/piper_model_exporter.ipynb) will allow you to do that. The export notebook also includes a link to another one that will allow you to [test your generated model](https://colab.research.google.com/github/rmcpantoja/piper/blob/master/notebooks/piper_inference_(ONNX).ipynb) by typing in text.

## Uploading To Drive

Before opening the notebook, zip up your audio files into a folder and upload them to Google Drive. If you are unfamiliar with this process, you may need to do some research online based on the platform you're using, as I can not provide assistance for every environment someone might happen to be running. Note that there is a desktop app to make this process easier by simply copying files from File Explore or Finder directly to Google drive, so you may wish to consider this option. At least on Mac, it is likely that additional files will be created when you zip up your folder. To prevent this from happening, open your folder of audio files, select all of them with command a, then right click and choose compress X number of files from the menu that appears.

## Using The Training Notebook
### Installing Software

All of these notebooks are well designed and self-explanatory for the most part, so I will give a high-level overview of what you will need to do in order to train and test a model. When you open the training notebook, you will need to install some dependencies. Everything is split up by headings, so it's quite easy to navigate, if a little cluttered.

At the top you will find a few cells to prevent Colab from disconnecting prematurely, as well as checking which GPU you have been assigned. It is up to you as to whether you wish to run these, and it will not adversely affect training if you don't. After this, you will find a cell to Mount your Google Drive to store model checkpoints. Next you will find a cell to install all necessary software for piper to run correctly. Simply click on the `Run Cell` button to the left of the installation section, and grant permission for the notebook to run when prompted. It is also wise to save a copy of this notebook in your drive in case changes are made later which break compatibility. You can do this from the file menu in the menu bar at the top.

While the cell runs, you should see output nearby that looks like a terminal. You are essentially using a Linux virtual machine under the hood that is hosted on Google servers, so if you're familiar with this OS a lot of the commands will be similar. cell output is contained in a frame next to each section, and before it you will see whether the cell is currently executing or has been executed. In the most recent updates to `colaboratory`, you can differentiate whether a cell is running and has finished running, through the `more_horiz` and `output` flags. Remember that to navigate between frames, the quick navigation key is "m". Wait until you see that the cell has finished running before proceeding to the next section.

### uploading Files

After getting everything installed, it's time to upload your data set. Enter the path to your .zip file in the next cell, and click run. To do this, go to the top of the page, navigate between buttons with the "b" quick navigation key until you find the "files" button, it's a collapsed button, so we need to expand it. Once done, we can find a new "files" header. By browsing, you can explore the files and folders on the machine. Folders are opened with the enter key. So, we must open "Drive", then "MyDrive", and once done, find the folders and subfolders until we find the .zip file of the audios. Once positioned in the file, we need to set this object to focus on the mouse cursor (you can check the keyboard reference of your favorite screen reader), and finally right click. Next, a menu opens, so we select the "copy path" item, we again focus this item on the mouse cursor and we can press it using the left click. Then, you can go back to the content of the cell to upload your audios and paste the path that has been copied. After this, upload your .csv file with the transcript. When uploading the transcription, find the upload button that will be located in the output frame of the relevant cell, and click it. Select the file using your OS file browser, and upon pressing upload the file will be saved immediately.

### Pre-Processing And Configuring Training Settings

After everything has been successfully uploaded, it is time to pre-process your data set. Please be sure to look at every parameter carefully before clicking the run cell button. Among other things, you will be asked to choose a name for your model, the language, and the sample rate of your data. Note that for English, I would recommend choosing US English for now, as this will affect the availability of pre-trained models to fine-tune from, and currently LJ speech produces the best results in my testing. If you're training UK English, this will lead to some incorrect pronunciation, so feel free to experiment with British English if you wish.

After the pre-processing step has completed, you must configure various training settings for your model. You can leave most of these at the defaults. In the action combobox, we have a few possibilities, such as `Continue training`, `fine-tuning`, `training multiple speakers` using a single speaker model, and `training from scratch`. In this case, we will use fine-tuning.

Another parameter to look at is the quality setting. In all of my tests so far, I have said this to medium, but you can experiment with other quality levels if you wish.

And finally, something to keep in mind is the batch size. This depends on the dataset and GPU resources you have. Colab by default offers a 16 GB tesla t4, but you can purchase colab pro to train larger datasets for a longer run time. Below is a table with different dataset sizes and suggested values:

| Dataset size | suggested batch size |
|:---:|:---:|
|>50 audio-text pairs|6|
|>100 audio-text pairs|8|
|>200 audio-text pairs|12|
|>250 audio-text pairs|14-16|
|>500 audio-text pairs|18-20|
|>1000 audio-text pairs|20-24|
|>5000 audio-text pairs|24-32|
|>10000 audio-text pairs|48-64|

Please note that these are only suggested values, but depend on resources such as GPU VRAM and available compute.

After clicking the run button on this cell, if you selected the `fine-tune` or `resume from a single-speaker model` action, in the output you will see a drop-down menu to select the model you wish to fine-tune. Be sure to select a model with the same quality level that you chose before in order for everything to work correctly.

Note: to continue a training, it can only be done once and, in addition, the same settings and parameters must be set as those that were applied the first time it was trained.

### Training

All right, finally it's time to train! If you did everything right, clicking on the run button for the training cell will begin the process. Depending on the number of Epochs you have set, training may not stop by itself, but once you have progress saved in Drive you can interrupt it at any time by clicking the run button again.

Note: if when you start training you get a `Cuda out of memory` error and you use our suggested batch size values, you can reduce this value a little by geting back to the settings cell. Modify the respective control and runn this cell to update these settings, then try training again.

To listen to how the model sounds while training, you can do so by accessing some audios that will appear in your `Google Drive` and will be constantly updated. The amount of time to train will depend on how much data you have, but I've gotten good results after about three hours or so. Feel free to experiment with this.

## Exporting and Testing

Model files are saved in the working directory you specified earlier in this process. Under `Lightning_logs` you will find a checkpoints folder that will contain a `last.ckpt` file as training progresses, and in the path of your model folder you will also find a config.json that you will need in order to export the finished voice.

Similar to the training notebook, the export and testing notebooks require you to install software before running them. Both of these notebooks include an accessibility feature to provide voice prompts as input from you is needed, and this is very helpful to keep you from having to constantly check the output. Everything is quite self-explanatory here, however I will briefly talk about creating links to your model files in order to export them. In Google Drive, you must create shareable links that can be viewed by everyone to pass into both notebooks, and this process will differ depending on the platform you are using. On the web, you can right click on a file and choose "copy link". Make sure that you choose "manage access" and change permission to "anyone with the link". Do this for both your checkpoint and configuration files, and paste them into the relevant text fields. Fill in the other parameters as you did for the training notebook, and click run. Creating a model card is optional. A model card is simply a text file that explains information such as the sample rate and the data set you used, but you don't need to fill this in if you don't want to. After the model has been created, in the next cell, you can choose how you want to export the dataset, either by uploading to your Google Drive or downloading to your device (may take a while).

If you don't use Windows or have NVDA but want to play around with your model and any others that people might share with you or that you find elsewhere, open up the [testing notebook](https://colab.research.google.com/github/rmcpantoja/piper/blob/master/notebooks/piper_inference_(ONNX).ipynb). This is very similar to the export notebook, in that you install the software and copy a link to your .tar file into the box provided. Just like the model exporter, voice prompts can be enabled to make the process a little easier. You also have control over the speech rate and a few parameters relating to variation. Feel free to play around with this as long as you like. Note that although the input text box is not multi line, you can enter or paste any text you wish from the clipboard, and Piper will read it for you.

# Conclusion

So that's it, you've trained your first Piper model. While the process does seem quite involved at first glance, as you go through I think you'll find that it's not as hard as you might imagine. If you have any questions or wish to make corrections to the guide, please don't hesitate to submit an issue or PR and I will get back to you ASAP. Thanks for reading, and I can't wait to see what you do with this technology. Happy training!
