# How it worked for me
I've found no documentation how to use the entropia libsocket-can-java JNI-Library, so my solution (running with MCP 2515 module) written down:

Get SocketCan running on the Pi, so it shows up "can0" interface in `ifconfig`. You will find a tutorial for this. If it is running, do the following:

1. Clone the libsocket-can-java Library to the Raspberry Pi 3 with `git clone` (There is a Makefile in the cloned directory. It contains all info to compile the C Library as an .so "Shared Library")
2. Compile it with commandline `make` (This requires at least a C compiler like gcc) -> Notice my advice 'JDK Version'
3. In the folder the file `lib/libjni_socketcan.so` was created. Copy this shared library file to `/usr/lib`. Linux and the Java programm will find it there now.
4. Create a Java program to test it.

In Java you import the `CanSocket.java` class to your project. An example how you can use it:

## JDK Version and compile errors with javah
In JDK 10 the `javah` compiler was removed. It was set deprecated in JDK 9(?) and replaced by using `javac -h`.
**I changed the Makefile (line 60-63) to work with the `javac -h` because i'm using JDK 11 now. If you have trouble with it/using JDK 8, comment in the other line for your JDK version.**

## Linux kernel v5
After switching to the kernel v5.10 on Raspberry the error 'java.lang.IllegalArgumentException: illegal AF_CAN address' occured on the same hardware and Java setup. The problem was caused by old include files for compiling. 
The include files in this repo are a copy of some files in the linux repo:
```
linux\include\uapi\linux\can.h
linux\include\uapi\linux\can\*
```

I replaced them with the new ones and after compiling it, it seem to work now again, see commit with tag v5.10.

*If such issues happen again in future: Try to do the same I explained above, replace the include files with the matching version of the linux files and try it.*

## Java Example
```java
public class Main {
    public static void main(String[] args) {
        System.out.println("Start...");

        try {
            // Init
            CanSocket socket = new CanSocket(CanSocket.Mode.RAW);
            CanInterface canInterface = new CanInterface(socket, "can0"); // Must be the name of the can interface you find in ifconfig
            socket.bind(canInterface);

            // Reading
            for (int i = 0; i < 5; i++) {
                System.out.println(Arrays.toString(socket.recv().getData())); // Will hang on if there is no data to receive.
                sleep(100);
            }

            {
                // Sending Standard Format
                byte[] data = {0x02};
                int address = 0x1F;
                CanFrame frame = new CanFrame(canInterface, new CanSocket.CanId(address), data);
                System.out.println("Sending frame... Addr: " + address + " Data: " + Arrays.toString(data));
                socket.send(frame);
            }

            {
                // Sending Extended Format - notice setEFFSFF()
                byte[] data = {0x01};
                int address = 0x56CCCCC;            
                CanFrame frame = new CanFrame(canInterface, new CanSocket.CanId(address).setEFFSFF(), data);
                System.out.println("Sending frame... Addr: " + address + " Data: " + Arrays.toString(data));
                socket.send(frame);
            }

            // Close the socket. Maybe optional
            socket.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private static void sleep(int ms){
        try {
            Thread.sleep(ms);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

## Notice
I can't serve any help and won't improve it, I am not working on it - I only use it and wanted to share my way how to get it work.

*Keywords: Socket CAN Java, JNI, Linux, Raspberry Pi, Raspbian, ARM*
