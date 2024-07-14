https://stackoverflow.com/questions/1360632/what-are-object-serialization-and-deserialization
- serialization refers to creating a version of our data(rather than the entire object) that can be used for storage, which is usually is string or as raw binary
	- what is the difference between object and data?
	- why is string, an option for serialization?
		- so that humans can also understand?
	- an object does not just have data associated with it, it has methods as well
https://stackoverflow.com/questions/447898/what-is-object-serialization
- everything in computer is bytes, no denying that
- those bytes are scattered around
- a document we might be typing may have text in one place and formatting in another or even each line of text in different places
- serialization collects all stuff relevant to what we are serializing into a linear sequence of bytes
	- ex: storing an object reference is of no use, serialization goes to the relevant data and serializes it
- possibly with extra conversion, for example to make it platform independent
- when we implement serialization, we may skip to avoid certain parts of data being serialized
- second answer also is good
- read third answer
- 