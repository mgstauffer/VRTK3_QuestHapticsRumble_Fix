# VRTK3_QuestHapticsRumble_Fix
A simple fix/workaround for the weak haptics/rumble on Quest when using VRTK3 / VRTK3.3

My project uses
VRTK 3.3
Oculus SDK 1.31
Unity 2018.4.9

If you're choosing to use (or stuck using for the time being, like me) VRTK 3.3 (VRTK3, VRTK3.3) in your Unity project for Quest development, you probably noticed that the haptic/rumble response is very weak when using the VRTK `VRTK_ControllerHaptics.TriggerHapticPulse` methods (and probably if you call the OVR methods direclty, but I haven't tried).

This repo has just a simple patch file to fix/workaround this problem, and create stronger rumble. You can still change the strength of the rumble using `VRTK_ControllerHaptics.TriggerHapticPulse` to get a range of rumble intensity.

If you don't want to use the patch file, here's the code, which replaces `OVRHapticsClip.WriteSample()` 

```
    /// <summary>
    /// Adds the specified sample to the end of the clip.
    /// </summary>
    public void WriteSample(byte sample) // TODO support multi-byte samples
    {
        if (Count >= Capacity)
        {
            //Debug.LogError("Attempted to write OVRHapticsClip sample out of range - Count:" + Count + " Capacity:" + Capacity);
            return;
        }

        //WORKROUND
        //Haptics strength on quest controllers is VERY weak with this version of OVR,
        // at least when using VRTK3.3 and Unity 2018.4.
        //Note that the trouble also happened when using quest via link and running thru
        // unity editor and Oculus desktop/rift app.
        //This workaround fills the whole clip (ie 'Samples') with 'sample' and yields a much stronger
        // haptic response. Otherwise only the first element in 'Samples' gets filled and the rest are 0
        //This hasn't been tested extensively, but it does work to set different strength of haptic response
        // by changing 'sample'.
        if (OVRHaptics.Config.SampleSizeInBytes == 1)
        {
            //NOTE - Array.Fill is not available in this version of C#
            for (int i = 0; i < Capacity; i++)
            {
                Samples[i] = sample;
            }
            Count = Capacity;
        }
        // END WORKAROUND

        //orig
        /*
        if (OVRHaptics.Config.SampleSizeInBytes == 1)
        {
            Samples[Count * OVRHaptics.Config.SampleSizeInBytes] = sample;
        }
        Count++;
        */
    }
```
