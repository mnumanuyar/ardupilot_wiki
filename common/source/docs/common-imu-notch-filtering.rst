.. _common-imu-notch-filtering:

[copywiki destination="copter,plane"]

===========================================================
Managing Gyro Noise with the Dynamic Harmonic Notch Filters
===========================================================

As discussed under the :ref:`Vibration Damping<common-vibration-damping>` topic, managing vibration in ArduPilot autopilot installations is extremely important in order to yield predictable control of an aircraft. Typically installations utilize mechanical vibration damping for the autopilot, internally or externally, in order to remove the worst of the vibration. However, mechanical damping can only go so far and software filtering must be used to remove further noise.

To the autopilot, vibration noise looks like any other disturbance (e.g. wind, turbulence, control link slop, etc.) that the autopilot must compensate for in order to control the aircraft. This prevents optimum tuning of the attitude control loops and decreased performance.

For multicopters and QuadPlanes, virtually all vibrations originate from the motor's rotational frequency.  For helicopters and planes, the vibrations are linked to the main rotor/prop speed.

ArduPilot has support for two notch filters whose filter frequency can be linked to the motor rotational frequency for motors, or the rotor speed for helicopters, and provides notches at a primary frequency and its harmonics. 

The dynamic notch is enabled overall by setting :ref:`INS_HNTCH_ENABLE<INS_HNTCH_ENABLE>` = 1 for the first notch, and :ref:`INS_HNTC2_ENABLE<INS_HNTC2_ENABLE>` = 1, for the second. After rebooting, all the relevant parameters will appear.

Determining Notch Filter Center Frequency
=========================================

Before actually setting up a dynamic notch filter, the frequencies that are desired to be rejected must first be determined. This is crucial if :ref:`common-throttle-based-notch` is used. While the other methods do not require this knowledge apriori, its still worthwhile as a comparison point for the post filter activation analysis of the filter's effectiveness

See the :ref:`common-imu-batchsampling` page for this step. Once the noise frequency is determined, the notch filter(s) can be setup.

Notch Filter Control Types
==========================

Key to the dynamic notch filter operation is control of its center frequency. There are five methods that can be used for doing this:

#. :ref:`INS_HNTCH_MODE <INS_HNTCH_MODE>` = 0. Dynamic notch frequency control disabled. The center frequency is fixed and is static. Often used in Traditional Helicopters with external governors for rotor speed, either incorporated in the ESC or separate for ICE motors.
#. :ref:`INS_HNTCH_MODE <INS_HNTCH_MODE>` = 1. (Default) Throttle position based, where the frequency at hover throttle is determined by analysis of logs, and then variation of throttle position above this is used to track the increase in noise frequency. Note that the throttle reference only applies to VTOL motors in QuadPlanes, not forward motors, and will not be effective in fixed wing only flight modes. See :ref:`throttle-based<common-imu-notch-filtering-throttle-based-setup>` for further setup details.
#. :ref:`INS_HNTCH_MODE <INS_HNTCH_MODE>` = 2. RPM sensor based, where an external :ref:`RPM sensor <common-rpm>` is used to determine the motor frequency and hence primary vibration source's frequency for the notch. Often used in Traditional Helicopters (See :ref:`Helicopters<common-imu-notch-filtering-helicopter-setup>`) using the ArduPilot Head Speed Governor feature. See :ref:`RPM Sensor<common-rpm-based-notch>` for further setup instructions.
#. :ref:`INS_HNTCH_MODE <INS_HNTCH_MODE>` = 3. ESC Telemetry based, where the ESC provides motor RPM information which is used to set the center frequency. This can also be used for the forward motor in fixed wing flight, if the forward motor(s) ESCs report RPM. This requires that your ESCs are configured correctly to support BLHeli telemetry via :ref:`a serial port<blheli32-esc-telemetry>`. See :ref:`ESC Telemetry<common-esc-telem-based-notch>` for further setup instructions.
#. :ref:`INS_HNTCH_MODE <INS_HNTCH_MODE>` = 4. If your autopilot supports it (ie. has more than 2MB of flash, see :ref:`common-limited_firmware`), In-Flight FFT, where a running FFT is done in flight to determine the primary noise frequency and adjust the notch's center frequency to match. This probably the best mode if the autopilot is capable of running this feature. This mode also works on fixed wing only Planes. See :ref:`In-Flight FFT <common-imu-fft>` for further setup instructions.

All of the above are repeated, independently, for the second notch and are prefaced with ``INS_HNTC2_`` instead of ``INS_HNTCH_``. The following will explain setup for the first set of notches.

Number of Harmonics Filtered
============================

- Set :ref:`INS_HNTCH_HMNCS <INS_HNTCH_HMNCS>` to enable up to three harmonics (multiples of the center frequency) for notches.

Checking Notch Filter Effectiveness
===================================

Once the notch filter(s) are setup, the effectiveness of them can be checked by again measuring the  frequency spectrum of the output of the filters (which are the new inputs to the IMU sensors). Refer back to the :ref:`common-imu-batchsampling`  page for this.

Double-Notch
============

The software notch filters used are very "spikey" being relatively narrow but good at attenuation at their center. On larger copters the noise profile of the motors is quite dirty covering a broader range of frequencies than can be covered by a single notch filter. In order to address this situation it is possible to configure the harmonic notches as double notches that gives a wider spread of significant attenuation. To utilize this feature set :ref:`INS_HNTCH_OPTS <INS_HNTCH_OPTS>` to "1".

.. note:: Each notch has some CPU cost so if you configure both dynamic harmonics and double notches (:ref:`INS_HNTCH_OPTS <INS_HNTCH_OPTS>` set to 3) you will end up with 8 notches on your aircraft per IMU. On flight controllers with 3 IMUs, this totals 24 notches which is computationally significant and could impact operation. For example, with F4 cpus with one IMU, using :ref:`INS_GYRO_RATE<INS_GYRO_RATE>` =0 (1khz) this is safe, as is 3 IMUs running with fast sampling (:ref:`INS_GYRO_RATE<INS_GYRO_RATE>` =1 (2khz) on H7 cpus.

Note also that with a double-notch the maximum attenuation is either side of the center frequency so on smaller aircraft with a very pronounced peak their use is usually counter productive.

.. toctree::
    :hidden:

    Throttle Based <common-throttle-based-notch>
    RPM Sensor<common-rpm-based-notch>
    ESC Telemetry<common-esc-telem-based-notch>
    In-Flight FFT <common-imu-fft>
    Traditional Heli Notch Filter Setup<common-imu-notch-filtering-helicopter-setup>

