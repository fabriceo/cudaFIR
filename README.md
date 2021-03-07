# cudaFIR
cudaFIr is an ALSA audio plugin that implements Finite impulse response (FIR) filtering.
It convolves the input stereo signal with the filter impulse response, using partitionned FFT, accelerated by Nvidia GPU.

## Compile

cudaFIR compilation need libasound-devel, make, gcc, CMake packages and the cuda SDK from Nvidia.

To compile it : 
```
cd cudaFIR
Edit the CMakeLists.txt file to set the CUDA_ARCHITECTURES of your GPU
mkdir build
cd build
cmake ..
make 
```

This will create the `libasound_module_pcm_cudaFIR.so` alsa external pluging in the build directory

Note : cudaFIR has been only tested on a Jetson TX1, but it must works on any Linux machine with a Nvidia GPU for which the cuda SDK exist.

## Install & Alsa configuration

1. Copy the libasound_module_pcm_cudaFIR.so into your alsa external plugin directory. Exact path depends of your Linux distribution (ie for fedora : /usr/lib64/alsa-lib/).

2 . Create a virtual alsa device in your alsa conf file (.asoundrc, /etc/asound.conf, etc ...) :

```
pcm.filter {
        type cudaFIR
	filterpathprefix "/mydir/myfilter"
        slave {
                pcm "hw:USB,0"
        }
}
```

* `hw:USB,0` must be replaced by the name of your alsa output device.
* `filterpathprefix` is the filters filename prefix (see below).

### Filters
cudaFIR need one filter impulse response file by used sample rate.
Filters impulse response file format is 32bits floats stereo raw files.
Their names must be of the form: `filterpathprefix`-Fsk.raw , where Fs is the sample frequency in Ks/s.
Example  : /mydir/myfilter-44k.raw, /mydir/myfilter-48k.raw, /mydir/myfilter-88k.raw, /mydir/myfilter-96k.raw, etc ...

## Use

Just play music using the `filter` virtual alsa device.

Example:
aplay -D filter sound.wav

Note : cudaFIR accept only pcm signed 16 or 32 bits integer as input, and output in 32 bits signed integer format, so the slave device must accept this later format.
