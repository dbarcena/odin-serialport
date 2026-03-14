# serial

This project is based on the project `github.com/jasonKercher/serial`, which is a wrapper for serial communication on Windows and Linux. The original project did not compile with the Odin version `dev-2026-03-nightly:6d9a611`, so I created this fork to adapt it to my needs and to serve as practice for using external C libraries in Odin.



## INSTALL AND USE

Clone the repository into the `vendor` folder of your project `git clone https://github.com/dbarcena/odin-serial.git vendor/serial`


```odin
import "vendor/serial"
```

## serial_open

Opens the serial port, taking a pointer to a Serial structure, the port name, the baud rate, and the number of data bits. Returns true if the port was opened successfully or false if an error occurred.

```odin

	s := serial.Serial

  if serial.serial_open(&s, "COM3", serial.Baud_Rate.B115200 ,8)  {
    log.info("Open serial port.")
  } else {
    log.error("Failed to open serial port.")
    return
  }
  defer serial.serial_close(&s)

```

## serial_close

Closes the serial port.

```odin
  serial.serial_close(&s)
```

## serial_queryRecv

Checks whether data is available to read without blocking execution. Returns the number of bytes available, or -1 if an error occurs.

```odin
  n: int = serial.serial_queryRecv(&s, buf[:])
  log.info("Bytes available to read: %d", n)
```

## serial_recv

Retrieves as much available data as can fit into the provided buffer. Returns the number of bytes read or -1 if an error occurs.

```odin
  var buf: [64]u8
  n: int = serial.serial_recv(&s, buf[:])
  log.info("Read %d bytes.", n)
```

## serial_send

Sends the specified number of bytes from the provided buffer. Returns the number of bytes written or -1 if an error occurs.

```odin
  data: [5]u8 = [5]u8{1, 2, 3, 4, 5}
  n: int = serial.serial_send(&s, data[:])
  log.info("Written %d bytes.", n)
```

# serialasync

It is a functionality that encapsulates `serial` to perform read and write operations asynchronously using threads and channels. This allows the program to continue running while serial communication operations are performed in the background.

## serialasync_open

Opens and creates the transmission and reception channels. Returns true if opened successfully or false if an error occurred.

```odin
	sa: serial.SerialAsync

	if serial.serialasync_open(&sa, "COM3", serial.Baud_Rate.B115200, 8) {
		log.info("Open serial port.")
	} else {
		log.error("Failed to open serial port.")
		return
	}
	defer serial.serialasync_close(&sa)
```

## serialasync_close

Closes the serial port and the transmission and reception channels.

```odin
  serial.serialasync_close(&sa)
```

## serialasync_send

Add a message to the transmission channel to be sent by the serial communication thread. Returns true if the message was successfully added to the channel or false if an error occurred.

```odin
  data: [5]u8 = [5]u8{1, 2, 3, 4, 5}
  if serial.serialasync_send(&sa, data[:]) {
    log.info("Message sent to async channel.")
  } else {
    log.error("Failed to send message to async channel.")
  }
```

## serialasync_async

It should be called from a thread if you want the serial port I/O to be managed asynchronously. The thread will remain active as long as the serial port is open, handling the reading and writing of data through the corresponding channels.

```odin
	consumer_thread := thread.create_and_start_with_poly_data(&sa, serial.serialasync_async)
	if consumer_thread == nil {
		log.error("Failed to create consumer thread")
		return
	}
	defer thread.destroy(consumer_thread)
```

## serialasync_stop

Add a signal so that if there is a thread running `serialasync_async`, it stops safely.

```odin
  serial.serialasync_stop(&sa)
```

## serialasync_read_frame

If you have a thread running `serialasync_async`, you can read the data received through the reception channel using this function, which takes a delimiter to indicate the end of a message. It returns the data read as a string or an empty string if no message could be read.

```odin
	for {
		data := serial.serialasync_read_frame(&sa,"\r\n")
		if len(data)==0 {
      break
    }
		log.infof("Data=%s", data)
	}
```
