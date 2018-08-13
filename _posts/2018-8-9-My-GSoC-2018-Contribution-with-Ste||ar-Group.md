Being interested in distributed systems, I was interested in working on that project. The project is about adding all-to-all communications to HPX library. It was a tough project. It was a very good experience but it could have been much better. Anyway, let's have a look on my contribution in HPX library with Ste\|\|ar Group in GSoC 2018.

## Main Goal of the project

Building a libfabric backend with the model proposed by the [FFLIB](https://spcl.inf.ethz.ch/Research/Parallel_Programming/FFlib/) library. Afterthat, that backend layer would be integrated with the libfabric parcelport layer developed in [HPX](https://github.com/STEllAR-GROUP/hpx/tree/master/plugins/parcelport/libfabric) to introduce collectives communications to HPX.

## Starting with FFLIB library

My first task was to understand the [FFLIB1](https://spcl.inf.ethz.ch/Research/Parallel_Programming/FFlib/) code. That version was developed to demonstrate the FFLIB model on Portals4 layer. So my goal was to understand the code to get the idea how to tweak that to work on libfabric instead of Portals4. I had to go into the [documentation of Portals4](http://www.cs.sandia.gov/Portals/portals40.pdf) to understand its structure and APIs. It took me two/three weeks to understand the FFLIB1 code and the Portals4 APIs used in the code. The FFLIB1 code had no documentation so it was a difficult initial task. Afterthat, we made a call with Salvatore (the author of FFLIB) and decided that we should go with [FFLIB2](https://github.com/SalvatoreDiGirolamo/fflib2). That version was in a very early state. It had a small structure and can do send/recv functionalities. It was working on MPI. We decided to build a libfabric component in that version as it would be easier than tweaking the first version.

## Building a simple libfabric test

There was a simple libfabric pingpong [example](https://github.com/ofiwg/libfabric/blob/master/util/pingpong.c) developed by [OFIWG](https://github.com/ofiwg/ofi-guide/blob/master/OFIGuide.md). So my first part was to get this example working with FFLIB. It was a simple code passing messages like pingpong between 2 nodes. The example worked on sockets so I could run it successfully on my laptop.

First I worked on dissecting the code into separate parts. Each part was doing a separate job so that I can understand it. It was divided into 3 sections:

The first section was the configuration part. It was about taking arguments from the user to configure how the test should be working. One of the parameters was the connectivity type between the 2 nodes. The user can decide whether it will be connectionless or connection oriented test. Another parameter was the transfer size i.e, How much data would be transferred between the 2 nodes. Another parameter was the number of ping-pong iterations. There were many parameters which the user can use to tweak the test.

The second section was about initializing the test. For example, initializing the connection between the client and the server in the case of connection oriented test. Also allocating the needed resources for the test such as: fabric, domain, endpoints, memory regions, completion queues and other resources that will be used by each node.

The last section was the actual send/receive part. It initializes the data to be sent/received every iteration and posts the data transfer operation.

I understood the implementation of every section and then used the APIs in this test to build my own test with FFLIB. I forked the FFLIB repo and implemented a libfabric FFLIB component which is linked with the libfabric APIs used in the pingpong test. Then I built a send/receive test in FFLIB that worked with this component. There were problems with compiling and linking the files because of some includes that were used in the pingpong file but weren't put in the include directory of libfabric.

At that time, John sent me another libfabric connection test that was implemented by Thomas Heller. It was doing the connection between 2 nodes on sockets and needed only some additional tweaks so that it can be used in a ping-pong test. So I worked on building and understanding this test while John was working on the compilation and linking errors in the other test.

Thomas's test code was very understandable and it was more organized so It didn't take me much time to understand it. I used it to build another test for FFLIB with libfabric. There were some linking problems because Thomas's test used CPP and FFLIB is a C library but with the help of John everything went well.

Finally I managed to build a FFLIB test working on libfabric. After the build process, there were some logical errors in my code. It took me some time to debug the errors but I managed to fix them.

## The next step

At that point we have a FFLIB test working on libfabric. It's a simple test but it's a basic start for more complex implementations.

Then we had to decide in which track we should continue. We had two options: Integrating that simple test with the [HPX parcelport](https://github.com/STEllAR-GROUP/hpx/tree/master/plugins/parcelport/libfabric) code or implementing more complex collective communications tests in the FFLIB libfabric backend. We chose to integrate that simple test first with the parcelport.

Unfortunately, the parcelport code also had no documentation. John helped me to get started with the code but it was too difficult to understand. It had many terminologies that were very confusing. I spent two weeks trying to figure out how should I modify the code to integrate the FFLIB stuff with it. But I got stuck many many times. I had calls with John but it didn't help me much because the code needed documentation. It was impossible for me to understand it without a good documentation.

So we decided that John would go with that integration part because it would be easier for him and I would go with building more complex tests in FFLIB.

At that time Salvatore had made some progress with the FFLIB code. It added some more stuff to the structure. So I started building a pingpong test working with the FFLIB schedule. I built it easily it wasn't a hard task and everything went well without problems. 

Afterthat I started building an all-reduce test. The all-reduce algorithm was implemented by Salvatore in FFLIB. I had to use it to build the test. After building that test, it had some errors. They were due to the changes made by Salvatore in the FFLIB. I had to modify some parts of my code to go with the new structure. Then after fixing these errors, the test compiled and run properly but it was generating wrong results. I contacted Salvatore to help me figure out what was going wrong. Till now I haven't fixed the test but I am working on it.

## Conclusion

What I've done in my [FFLIB2 fork](https://github.com/AhmedSa-mir/fflib2/tree/hpx-libfabric):
- Implementing a libfabric component in the FFLIB library.
- Building 3 tests working on 2 nodes:
  - send/recv test
  - pingpong_schedule test
  - allreduce test (I'll fix it even after the GSoC period)

I've made a [gist file](https://gist.github.com/AhmedSa-mir/f1fdcc8612df83e843d9a69fc2fb52bd) that contains some useful information about what I've done.

What's left:
- Integrating the tests with the HPX libfabric parcelport
- Extending the tests to work with many nodes not only two.
- Building more collective communications tests


## Final Words

At the end I can say that it was not a simple project. It needed to be more organized. I spent many weeks with tasks that we didn't benefit from. The Lack of documentations was also a big issue. I haven't seen a documentation of any of the codebases I have worked with throughout the project. That was very annoying and took me a lot of time to overcome.

My learning experience wasn't as expected due to the problems I've mentioned but I got some knowledge. I've read some papers and dealt with different layers and that was interesting. I've implemented a small backend layer that should be used to integrate the FFLIB stuff with HPX parcelport. It wasn't that big but it was a good experience to get into these problems.

I am planning to continue contributing with HPX. I liked the project inspite of all the problems and I hope I contribute to something big in the HPX library project.

## References

[1] [FFLIB model library and paper](https://spcl.inf.ethz.ch/Research/Parallel_Programming/FFlib/)

[2] [FFLIB2 library](https://github.com/SalvatoreDiGirolamo/fflib2)

[3] [My FFLIB2 fork](https://github.com/AhmedSa-mir/fflib2/tree/hpx-libfabric)

[4] [Gist file that contains some useful information about what I've done throughout GSoC period](https://gist.github.com/AhmedSa-mir/f1fdcc8612df83e843d9a69fc2fb52bd) 

[5] [HPX libfabric parcelport](https://github.com/STEllAR-GROUP/hpx/tree/master/plugins/parcelport/libfabric)

[6] [OFIWG libfabric guide](https://github.com/ofiwg/ofi-guide/blob/master/OFIGuide.md)

[7] [How to run OFIWG pingpong test](https://github.com/ofiwg/libfabric/blob/master/man/fi_pingpong.1.md)

[8] [Portals4 Documentation](http://www.cs.sandia.gov/Portals/portals40.pdf)
