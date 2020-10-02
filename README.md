# Chirp Sounder 2

This software can be used to detect chirp sounders and over-the-horizon radar transmissions over the air, and to calculate ionograms. The software relies on <a href="https://github.com/MITHaystack/digital_rf">Digital RF</a> recordings of HF. 

This is a new implementation of the <a href="https://github.com/jvierine/chirpsounder">GNU Chirp Sounder</a>, GNU Chirp Sounder 2 allows you to automatically find chirps without knowledge of what the timing and chirp-rate is. 

The software consists of several parts:
 - detect_chirps.py  # this is used to find chirps using a chirp-rate matched filterbank
 - find_timings.py # this is used to cluster detections and determine what chirp timings and chirp rates exist
 - calc_ionograms.py # this is used to calculate ionograms based on parameters
 - plot_ionograms.py # plot calculated ionograms

Version:
Tested on Python 2.7.

## Installation

You need to compile the chirp downconversion library, which is written in C.
```
make 
```
There is no packaging or other installation needed. You just run the scripts in place. 

## Usage:
1) Make a data capture with THOR (comes with <a href="https://github.com/MITHaystack/digital_rf">DigitalRF</a>), a USRP N2x0, a GPSDO, and a broadband HF antenna in a quiet location: 

```
thor.py -m 192.168.10.3 -d "A:A" -c cha -f 12.5e6 -r 25e6 /dev/shm/hf25 
```

I use use a RAM disk ring buffer to avoid dropped packets, but this is not necessary. The software will be okay with dropped packets.

```
# copy digital rf from ram disk to permanent storage:
while true; do rsync -av --remove-source-files --exclude=tmp*
--progress /dev/shm/hf25/cha /data_out/hf25/ ; sleep 1 ; done
```

2) configure by copying the example1.ini configuration file (make sure you have the right center frequency, sample-rate, data directory, and channel name)
```
[config]

# channel name for the digital rf recording
channel="cha"

# the sample rate of the digital rf recording
sample_rate=25000000.0

# the center frequency of the digital rf recording
center_freq=12.5e6

# the location of the digital_rf recording
data_dir="/data_out/hf25"

# detection
threshold_snr=13.0

# how many chirps can we at most detect simultaneously
max_simultaneous_detections=5

# how sparsely do we search for chirps (1 .. N) 1 is slowest, but the most sensitive
# every Nth block is analyzed 
step=10            

# how many samples per block are coherently integrated on chirp detection
n_samples_per_block=5000000

minimum_frequency_spacing=0.2e6
# what chirp rates do we look for
chirp_rates=[50e3,100e3,125e3,500.0084e3]

# this is where all the data files are produced in
output_dir="./chirp2"

# what is the range resolution of the ionograms
range_resolution=2e3

# what is the frequency step of the ionogram
frequency_resolution=50e3

# what is the range extent around the strongest echo that is stored
max_range_extent=2000e3

# how many threads are used when chirp downconverting
n_downconversion_threads=4
```

3) detect chirps on the recording. can be parallelized with MPI to speed things up if you have lots of CPUs
```
mpirun -np 48 detect_chirps.py configuration.ini
```

4) run find_timings.py to cluster together multiple detections of the same chirp to create a database of chirp timings
```
find_timings.py configuration.ini
```

5) run calc_ionograms.py to generate ionograms based on the timings that were found. Can be paralellized with MPI. Keep in mind that adding a lot of processes may be detrimental to performance, due to the 100 MB/s read requirement. If you have a slow disk, don't use too many processes here! Each MPI process is additionally multi-threaded, with the number of threads configured in the configuration file
```
mpirun -np 4 calc_ionograms.py configuration.ini
```

6) run plot_ionograms.py to create plots
```
python plot_ionograms.py configuration.ini
```


## Examples

<img src="./examples/examples00.png" width="60%"/>

<img src="./examples/examples01.png" width="60%"/>

<img src="./examples/examples02.png" width="60%"/>

<img src="./examples/examples03.png" width="60%"/>

<img src="./examples/examples04.png" width="60%"/>

<img src="./examples/examples05.png" width="60%"/>

## Links

You can also use your sound card and HAM radio to detect chirps using the <a href="https://www.andrewsenior.me.uk/chirpview">Chirpview</a> program.
