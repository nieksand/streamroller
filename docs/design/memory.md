* Incoming messages write direct to zstd encoder for stream. 
* Baseline assumption of 3x compression ratio.  Obviously completely input dependent.
* Memory management really about handing out buffers for zstd output
* Simple pool of fixed size segments.  (64 MB?).  
* Abstraction which implements io::Write, pulls from fixed pool, releases when file uploaded
* Pool per core?  Encoder per core?
