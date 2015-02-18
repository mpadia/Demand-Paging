# Demand-Paging
Implemented complete virtual memory system with system calls to handle program memory. Implemented VA to PA translation. Mimic the concept of demand paging with backing storage in RAM.

What exactly is our memory layout?

        ______________________________
       /                              \
       |                              |
       |                              |
       |                              |
       |       Logical Memory         |
       |       (Virtual Pages)        |
       |                              |
       | Pages 4096 and beyond        |
       |------------------------------|<-\   /|------------------------------|
       |                              |   |  ||                              |
       |                              |   |  ||   8 Backing stores           |
       |                              |   | / |                              |
       |                              |   |-  |                              |
       |      Global Free Memory      |   | \ | Pages 2048 - 4095            |
       |                              |   |  ||------------------------------|
       |                              |   |  ||    1024 frames (our frames)  |
       | Pages 1024 - 4095            |   |  || Pages 1024-2047              |
       |------------------------------|<-/   \|------------------------------|
       |                              |
       |                              |
       |      Kernel/System Memory    |
       |                              |
       | Pages 406 - 1023             |
       |------------------------------|
       |     HOLE (reserved)          |
       |                              |
       | Pages 160 - 405              |
       |------------------------------|
       |                              |
       |                              |
       |      Kernel/System Memory    |
       |                              |
       | Pages 25-159                 |
       |                              |
       |------------------------------|
       | Text/Data                    |
       |______________________________|




What is a page?
    - A page is a logical unit of memory (in our case 4096 bytes).

What is a frame?
    - A frame is a physical location in memory that is the size of a 
      page.
      

What are virtual addresses?
    - Virtual addresses are all addresses (2^32 - 1 addresses)
      including addresses of the physical memory and beyond that. 
      They are called virtual addresses because it is a logical
      address rather than a physical address. Basically if an address
      is used that is greater than your amount of physical memory
      (in our case 16M) then a page fault will occur and the physical
      backing store (disk) that holds the value at that address will 
      be read from and brought into a physical frame. Page tables are
      used to keep track of what logical addresses(pages) are resident
      in physical memory(frames).

Basically what we have here are several structures that keep up with
all of the "mappings".

Page tables
    - Map virtual addresses(pages) to physical addresses(frames).

Inverted Page Table AKA Frame Table (useful when writing data out)
    - Each frame in memory (in our case there are NFRAMES frames we
      can use) has a corresponding frame table entry that keeps up
      with what backing store and page within the backing store the
      frame maps to. When a frame needs to be written back out to
      memory we can immediately know where to write the data using
      this information.

Backing Store Maps (useful when reading data in)
    - Each backing store maintains a list of maps that map what
      process and virtual page number map to this backing store.
      Using this information we can find exactly what backing store
      a virtual address maps to and read that data into a newly 
      allocated physical frame.


Since our backing stores are actually in memory too we have the
following setup:

                ______________________________
               /                              \
               |                              |
               |                              |
               |                              |
            /->|       Logical Memory         |<-\
            |  |       (Virutal Pages)        |   |
            |  |                              |   |
            |  |                              |   |
            |  |------------------------------|   |
Page Tables |  |                              |   |  Backing Store Maps
            |  |                              |   |
            |  |                              |   | 
            |  |                              |   |
            |  |        Backing Stores        |<-/
            |  |                              |<-\
            |  |______________________________|  |
            |  |                              |  |
            |  |                              |  |  Inverted Page Table
            |  |                              |  |
            \->|     1024 Physical Frames     |<-/
               |                              |
               |                              |
               |                              |
               |                              |
               |______________________________|
               |                              |
               |                              |
               |      Kernel/System Memory    |
               |                              |
               |                              |
               |______________________________|
