*   speed of oscillating field is given by 2f/p, in rotations per sec
*   $K_v$ is velocity constant. used in the equation $v_{rpm} = K_v V$. If you give a motor 1V power supply, it should rotate with maximum $K_v$ RPM, assuming no load. This value determines the speed, and the commutator frequency of the ESc simply has to match that of the motor to keep up.
*   more windings -> less speed -> less Kv (but higher torque)
*   more poles -> lower speed -> lower Kv (but higher torque)
*   torque is dir propto current. current draw of a bldc is given by
    $$I = \frac{V_{supply} - EMF_{back}}{R}$$
*   bldc is measured by four numbers XXYY. XX is the diameter, YY is height
*   higher volume means higher torque capability, because a larger coil will have larger L
*   the thing to watch out for in a BLDC is the temperature. when using one, you should always keep a close watch on the motor temps. especially is it's a new project, new battery, or new operating conditions. if temperature is out of range, check the load vs current (because torque), and see if gearing needs to be changed.
*   in general, the voltage limit of an ESC is a hard limit. if you overshoot, it might refuse (if firmware allows it to check) or be damaged. however, a motor won't easily be damaged as long as you keep temperatures in check
*   [this is common for most motor types] the motor will draw as much current as it wants, not caring about itself, the battery, or the ESC. so ESC should be rated for the highest possible current draw you expect, not the nominal or average draw. as for the motor - it is again important to monitor the temperature
*   speed-torque have a linear graph, with torque inversely prop to speed (for a given voltage). if you exceed the rated torque of a motor, it goes from continuous torque regime to intermittent torque regime
*   since the power supply is alternating, torque ripple exists. this is an oscillation (or otherwise, a disturbance) in the torque output of a motor. this may not be desirable in some situations
*   you do not want the motor to stall, as this can lead to build-up of heat
*   two transforms called clarke and park transform let you talk solely about d-axis (direct, magnetic flux) and q-axis (quadrature, torque). this is FOC (field oriented control), and it offers a way for the controller to deal with these currents independently. advantages: improved torque and speed control, reduced torque ripple, increased efficiency, and better dynamic response.
*   if you set Iq = 0, and Id != 0, you'd get zero RPM. but the motor can hold its position


kv = 1000

24 volt supply

rpm = 60, eventually rpm = 0