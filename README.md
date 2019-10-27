# stand-predictor
Experiment with [Salesforce's CTRL model](https://github.com/salesforce/ctrl) by predicting a character's [stand](https://jojo.fandom.com/wiki/Stand)

## Models
The best version of this model so far can be downloaded here.
* https://www.dropbox.com/s/mw26x2t32ioxeq8/checkpoint?dl=1
* https://www.dropbox.com/s/j0zrowcnf9xny0o/model.ckpt-413071.index?dl=1
* https://www.dropbox.com/s/b3z60l8lkn3cg2x/model.ckpt-413071.data-00000-of-00001?dl=1
* https://www.dropbox.com/s/jwncnf349guyka1/model.ckpt-413071.meta?dl=1

## Generation
Using a temperature of 0.1 seems to produce the best results for the above models.

```python3 generation.py --model_dir training_utils/seqlen256_v1.ckpt --temperature 0.1```

Use labels `Stand User: ` in front of the input, with `Stand Name: ` appended at the end, this will generate a "stand" as if that character were a stand user. The model works for both fictional characters and real life people.

Here's an example using Spongebob using text from a fandom wiki.

```
Stand User: SpongeBob is a childish, joyful, sea sponge who lives in a pineapple with his pet snail Gary in the underwater city of Bikini Bottom. He works as a fry cook, a job in which he is exceptionally skilled. He attends boating school, though he has yet to receive a driver's license due to his inability to drive a boatmobile properly. SpongeBob is very good-natured and loves to hang out with his best friend Patrick. Stand Name: 
```

The output will be something like this

```
Pineapple Express allows him to travel through water by floating on its surface. When not using it, he can use it like an umbrella. It also enables him to float at great speeds thanks to its ability to absorb water.
```

More examples can be found in `examples.txt`.

## Training
`train.txt` was used to train the models. Create a tfrecord file from the training data.

```python3 make_tf_records.py --text_file train.txt --control_code Stand --sequence_len 256```

Then run this command to start training

```nohup python3 training.py --model_dir seqlen256_v1.ckpt --iterations 71 &```

During training, I started at increments of 50 iterations, then smaller increments until I noticed some overfitting (characters start getting stands that already exist with no variation). In my experience, the lowest-cost configuration to train the model without going out-of-memory was an AWS spot instance with at least 64GB memory (no GPU). I used r5n.2xlarge for training.

## Notes
* The training data was gathered by scraping text from the fandom wiki (urls can be found in `stands_with_user.ndjson`). I left out the "appearance" section because the model would focus only on the appeareance of the stand.
* If you like a stand and want it to add more to to the description, you can just enter `Stand Name: [text]` in the input and the model will add more information.
