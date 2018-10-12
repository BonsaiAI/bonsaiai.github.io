# Simulink Househeat

Simulink, developed by The MathWorks, is a graphical programming environment for modeling, simulating and analyzing multi-domain dynamical systems.

Please follow our step-by-step tutorial for using Bonsai's Simulink Toolbox to [**Train a Simulink Model with Bonsai**][1] if you want to use this example. The Inkling File is covered here, and the rest of the necessary steps are covered in that tutorial.

This example shows how to use Simulink to create the thermal model of a house. This system models the outdoor environment, the thermal characteristics of the house, and the house heating system. The objective for the Bonsai AI is to reach the desired temperature.

## Inkling File

Inkling is a declarative, strongly typed programming language specifically designed for artificial intelligence (AI). It abstracts away the world of dynamic AI algorithms that require expertise in machine learning and enables more developers to program AI. Please review our [**Inkling Guide**][2] to learn more about these topics.

###### Schema

```inkling
schema HouseheatState
    Float32 set_temp,
    Float32 room_temp,
    Float32 outside_temp,
    Float32 dTroom,
    Float32 heat_output,
    Float32 dTout
end
```

The `HouseheatState` schema names six records and assigns a type to them.

```inkling
schema HouseheatAction
    Float32{ 0.0:1:1.0 } heater_on
end
```

The `HouseheatAction` schema names a single record — `heater_on` — and assigns a constrained type to it.

```inkling
schema HouseheatConfig
    Float32 outside_phase
end
```

The `HouseheatConfig` schema names one record — `outside_phase` - and assigns a type to it.

###### Concept

```inkling
concept thermostat is classifier
   predicts (HouseheatAction)
   follows input(HouseheatState)
   feeds output
end
```
The concept, `thermostat`, takes input from the simulation model about the state of the temperature in the house (`HouseheatState` schema). It outputs to the `HouseheatAction` schema. This is the AI's next move in the game.

###### Simulator

```inkling
simulator simulink_sim(HouseheatConfig)
    action (HouseheatAction)
    state (HouseheatState)
end
```

The `simulink_sim` gets information from two schemas. The first schema, `action`, specifies the schema for actions within the simulation. The second schema, `state` contains the state of the simulation sent to the lesson.

###### Curriculum

```inkling
curriculum my_curriculum
    train thermostat
    with simulator simulink_sim
    objective match_set_temp
        lesson my_first_lesson
            configure
	        constrain outside_phase with Float32{0.0}
            until
                maximize match_set_temp
end
```

The curriculum, `my_curriculum`, trains the `thermostat` concept using the `simulink_sim`. This curriculum contains one lesson, called `my_first_lesson`. It configures the simulation, by setting constraints for the state of the simulator.

The lesson trains until the AI has maximized the objective named `match_set_temp`.


[1]: ../tutorials/simulink.html
[2]: ../guides/inkling-guide.html#overview