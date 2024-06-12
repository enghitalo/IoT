### Using UART in C on Linux

Interfacing with UART in C on a Linux system involves using the POSIX API to open the serial port, configure it, and perform read and write operations. Here’s a step-by-step guide:

#### 1. Including Necessary Headers

First, you need to include the necessary headers:

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <fcntl.h>
#include <errno.h>
#include <termios.h>
```

#### 2. Opening the UART Device

You can open the UART device using the `open` system call:

```c
int uart_fd = open("/dev/ttyUSB0", O_RDWR | O_NOCTTY | O_NDELAY);
if (uart_fd == -1) {
    perror("Unable to open /dev/ttyUSB0");
    exit(EXIT_FAILURE);
}
```

#### 3. Configuring UART Settings

Configure the UART settings such as baud rate, parity, stop bits, etc., using `termios` structure:

```c
struct termios options;

tcgetattr(uart_fd, &options); // Get the current options for the port

cfsetispeed(&options, B9600); // Set the baud rates to 9600
cfsetospeed(&options, B9600);

options.c_cflag |= (CLOCAL | CREAD); // Enable the receiver and set local mode
options.c_cflag &= ~PARENB;          // No parity bit
options.c_cflag &= ~CSTOPB;          // 1 stop bit
options.c_cflag &= ~CSIZE;           // Mask the character size bits
options.c_cflag |= CS8;              // 8 data bits

options.c_cflag &= ~CRTSCTS;         // Disable hardware flow control

options.c_lflag &= ~(ICANON | ECHO | ECHOE | ISIG); // Raw input mode
options.c_iflag &= ~(IXON | IXOFF | IXANY);         // Disable software flow control
options.c_oflag &= ~OPOST;                          // Raw output mode

tcsetattr(uart_fd, TCSANOW, &options); // Apply the settings immediately
```

#### 4. Writing to UART

Write data to UART using the `write` system call:

```c
const char *data = "Hello, UART!\n";
int data_len = strlen(data);
int bytes_written = write(uart_fd, data, data_len);
if (bytes_written < 0) {
    perror("Failed to write to the output");
}
```

#### 5. Reading from UART

Read data from UART using the `read` system call:

```c
char buffer[256];
int bytes_read = read(uart_fd, buffer, sizeof(buffer) - 1);
if (bytes_read < 0) {
    perror("Failed to read from the input");
} else {
    buffer[bytes_read] = '\0'; // Null-terminate the string
    printf("Received: %s", buffer);
}
```

#### 6. Closing the UART Device

Finally, close the UART device when you are done:

```c
close(uart_fd);
```

#### 7. Full Example

Here’s the full code wrapped together:

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <fcntl.h>
#include <errno.h>
#include <termios.h>

int main() {
    int uart_fd = open("/dev/ttyUSB0", O_RDWR | O_NOCTTY | O_NDELAY);
    if (uart_fd == -1) {
        perror("Unable to open /dev/ttyUSB0");
        return EXIT_FAILURE;
    }

    struct termios options;
    tcgetattr(uart_fd, &options);

    cfsetispeed(&options, B9600);
    cfsetospeed(&options, B9600);

    options.c_cflag |= (CLOCAL | CREAD);
    options.c_cflag &= ~PARENB;
    options.c_cflag &= ~CSTOPB;
    options.c_cflag &= ~CSIZE;
    options.c_cflag |= CS8;
    options.c_cflag &= ~CRTSCTS;

    options.c_lflag &= ~(ICANON | ECHO | ECHOE | ISIG);
    options.c_iflag &= ~(IXON | IXOFF | IXANY);
    options.c_oflag &= ~OPOST;

    tcsetattr(uart_fd, TCSANOW, &options);

    const char *data = "Hello, UART!\n";
    int data_len = strlen(data);
    int bytes_written = write(uart_fd, data, data_len);
    if (bytes_written < 0) {
        perror("Failed to write to the output");
    }

    char buffer[256];
    int bytes_read = read(uart_fd, buffer, sizeof(buffer) - 1);
    if (bytes_read < 0) {
        perror("Failed to read from the input");
    } else {
        buffer[bytes_read] = '\0';
        printf("Received: %s", buffer);
    }

    close(uart_fd);
    return EXIT_SUCCESS;
}
```

This code opens a UART device, configures it for 9600 baud, 8 data bits, no parity, and 1 stop bit, writes "Hello, UART!" to it, reads any response, and prints it to the console.