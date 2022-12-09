# LSTM Music Generation ♫♪♪
Marlon Grandy, Justin Zheng, and Oliver Wang
## 1. Introduction
## 2. Dataset
We chose [this](https://www.kaggle.com/datasets/soumikrakshit/classical-music-midi) kaggle dataset of classical music midi data, scraped from [this](http://www.piano-midi.de) site containing piano midi files. In the entire file, there is 290 songs, 19 composers, and 550,000 notes in total. When training our model, we used  subset of 45 songs, 5 composers, and 75,000 notes with 277 unique notes and chords.
## 3. Preprocessing
## 4. Model Architectures
Throughout the progression of the project many different model architectures were tested. Initially, a single model was trained on just the note data. This model architecture resulted in a training accuracy of 43% and a validation accuracy of 37%. As the project progressed, and the corresponding note durations and offsets were added as data, the model architectures also progressed.

## 5. Model Performance
## 6. Complications
Over the course of this project we faced a few issues that we needed to work through:
### Early model performance issues
### Excessive note repetitions
Our network used to get into cycles where the same few notes were played over and over without any interesting variation. We were able to resolve this by adding diversity functions to our sampling which allowed for more interesting notes to be included.
### Generating off of a bad seed
Generally, our seeds are 64-note segments picked randomly from our dataset. At times, we could get a poor seed by picking the start or end of a song, or a particularly boring segment. When this occurs, our generated audio is impacted heavily.
### Strange sounding offsets
### Lack of RAM available in Google Colab
Unfortunately, we were limited by the technology we had available to us and were not able to train using the entire dataset. We then had to resort to using only a subset of the artists, which was still successful but not ideal.
## 7. Music Samples
Found below are samples of music that we were able to generate with our network. The first audio file is the seed that we fed the network, and the following one is the resulting output.

https://user-images.githubusercontent.com/82842273/206411747-683e7858-cc00-46b6-be91-ed5250be1665.mp4


https://user-images.githubusercontent.com/82842273/206411764-4d547ddf-cfd8-4b3f-902e-9ea94ede26a8.mp4


## 8. Closing Thoughts

