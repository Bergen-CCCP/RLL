# RLL
Robot Layout Language -- An EDSL for describing the hardware layout for a yampa robot

## Design Idea

The high level design idea is to create an EDSL much like [servant][servant], where
a type level EDSL is used to describe a tree. In servan't case, it is for web endpoints,
and type families are used to derive a warp server (or client). In our case, the idea is
to describe the hardware layout of a robot, and to use type families to derive the three
components of the `reactimate` function in yampa.

```haskell
reactimate :: Monad m => m a -> (Bool -> m (DTime, Maybe a)) -> (Bool -> b -> m Bool) -> SF a b -> m ()
```

So to explain this, `reactimate` takes in 4 arguments:
- `m a` -- The initial input to the signal function
- `Bool -> m (DTime, Maybe a)` -- Get an input to the singal function network. The `Bool` states weather is can block
- `Bool -> b -> m Bool` -- A function to update the state of the robot. The input `Bool` states weather 
   anything has changed, the return `Bool` is weather we should terminate.
- `SF a b` A signal function that describes the robot's behaviour

### Implementation Example Idea

```haskell

type Robot = SpikeR :<|> BuildHatR

type SpikeR = SpikePrime "/dev/ttyUSB0" :>
                (  Port 'A' :> Output SimpleLegoMotor Int
              :<|> Port 'C' :> Sensor LegoDistance Double
                )

type BuildHatR = LegoBuildHat "/dev/ttyUSB1" :>
                    ( Port 'A' :> InOut Servo Int Int
                    )

```

The library will provide the following:
```haskell

reactimateRobot :: HasRobot r => Proxy r -> InRobot r -> OutRobot r -> IO ()
```
Where `inRobot` and `outRobot` are type families that will be used to map a user provided
data structure to and from the hardware of the robot.


[servant]: https://www.servant.dev
