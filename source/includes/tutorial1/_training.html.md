# Train Your BRAIN

Now that you’ve written a curriculum of machine teaching (through your Inkling code) to connect your simulation to the Bonsai AI Engine, it’s time to prepare a new version for training. Since we previously created a new BRAIN for training, the `--brain` argument in the below commands is optional, but we’ve left it in to remind you which BRAIN the CLI is targeting.

```shell
# Push the edited files to the server
bonsai push

# Start a new BRAIN version
bonsai train start
```

Use [`bonsai push`][1] to upload your edited Inkling file to the server whenever you make changes (make sure you filled in the action and state types first or you will get an error!).

> Python 2

```shell
python move_a_point_sim.py --brain=move-a-point
```
> Python 3

```shell
python3 move_a_point_sim.py --brain=move-a-point
```

Once you have started training mode with [`bonsai train start`][2] it's time to start your simulation. Training will begin automatically after you connect your simulator.

## View your BRAIN training status

> ![Training BRAIN](../images/tutorial1-training.png)

View your BRAIN's training status as it trains on the simulator by going to the BRAIN's Dashboard page on [beta.bons.ai][3]. Training move-a-point takes about a minute to get sufficient training to find the goal quickly.

There is no automatic ending to training, you can train this brain for hours, but there will be diminishing returns after a few minutes because of how simple of a problem this is to solve. You should wait until the reward approaches and stabilizes around 18 (with the max being 20 if the AI is perfect every episode). This should only take about a minute.

## Stop Training

```shell
bonsai train stop
```

Once the BRAIN has gotten to this level of performance (or sooner if you prefer), CTRL-C to disconnect the simulator, then [`bonsai train stop`][4] will end the training, and proceed to prediction.

# Predict with Your BRAIN

> Python 2

```shell
python move_a_point_sim.py --predict=latest
```

> Python 3

```shell
python3 move_a_point_sim.py --predict=latest
```

After your BRAIN is finished training you can use it to move to a point as quickly as it can. How well it does depends on how long you let it train! Using your BRAIN involves starting your simulation, but now in prediction mode with `--predict=latest` which will use the version of the latest training session that you just ran.

> ![Predicting BRAIN](../images/tutorial1-predicting.png)

Now you can see how fast the simulation can move to a point depending on how far away from the point it started. The higher the distance the more steps the agent will likely take to reach the point.

And that’s it! You have now successfully learned how to test out a simulation, write your own types to connect a BRAIN, train, and predict from that BRAIN! 

[1]: ../references/cli-reference.html#bonsai-push
[2]: ../references/cli-reference.html#bonsai-train-start
[3]: https://beta.bons.ai
[4]: ../references/cli-reference.html#bonsai-train-stop
