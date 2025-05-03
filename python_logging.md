# Python Logging Guide: Basic to Advanced

The Python `logging` module provides a flexible framework for emitting log messages from applications. This guide walks through its functionality from basic to advanced usage.

## Table of Contents

1. [Basic Logging](#basic-logging)
2. [Log Levels](#log-levels)
3. [Configuration Methods](#configuration-methods)
4. [Formatting Log Messages](#formatting-log-messages)
5. [Logging to Files](#logging-to-files)
6. [Multiple Handlers](#multiple-handlers)
7. [Loggers and Logger Hierarchy](#loggers-and-logger-hierarchy)
8. [Filter Usage](#filter-usage)
9. [Exception Information](#exception-information)
10. [Advanced Configuration](#advanced-configuration)
11. [Best Practices](#best-practices)

## Basic Logging

The simplest way to use logging is through the module-level functions:

```python
import logging

# Log messages at different levels
logging.debug("This is a debug message")
logging.info("This is an info message")
logging.warning("This is a warning message")
logging.error("This is an error message")
logging.critical("This is a critical message")
```

Output:
```
WARNING:root:This is a warning message
ERROR:root:This is an error message
CRITICAL:root:This is a critical message
```

Notice that the DEBUG and INFO messages don't appear because the default level is WARNING.

## Log Levels

The logging module provides five standard levels:

| Level | Numeric Value | Description |
|-------|---------------|-------------|
| DEBUG | 10 | Detailed information, typically of interest only when diagnosing problems |
| INFO | 20 | Confirmation that things are working as expected |
| WARNING | 30 | Indication that something unexpected happened, or may happen in the near future |
| ERROR | 40 | Due to a more serious problem, the software has not been able to perform some function |
| CRITICAL | 50 | A serious error, indicating that the program itself may be unable to continue running |

You can also define custom log levels if needed:

```python
import logging

# Define custom log levels
logging.addLevelName(15, "VERBOSE")
logging.addLevelName(25, "NOTICE")

# Create a method for the custom level
def verbose(self, message, *args, **kwargs):
    if self.isEnabledFor(15):
        self._log(15, message, args, **kwargs)

# Add the method to the Logger class
logging.Logger.verbose = verbose

# Configure logging
logging.basicConfig(level=logging.DEBUG, format='%(levelname)s: %(message)s')

# Use custom level
logger = logging.getLogger(__name__)
logger.verbose("This is a verbose message")
```

Output:
```
VERBOSE: This is a verbose message
```

## Configuration Methods

### Basic Configuration

The `basicConfig()` function provides a simple way to configure logging:

```python
import logging

logging.basicConfig(
    level=logging.DEBUG,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    datefmt='%Y-%m-%d %H:%M:%S',
    filename='app.log',
    filemode='w'
)

logging.debug("Debug message will be written to the file")
```

Output in app.log:
```
2025-05-04 12:34:56 - root - DEBUG - Debug message will be written to the file
```

### Dictionary Configuration

For more complex setups, you can use dictConfig:

```python
import logging
import logging.config

config = {
    'version': 1,
    'formatters': {
        'standard': {
            'format': '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
        },
    },
    'handlers': {
        'console': {
            'class': 'logging.StreamHandler',
            'level': 'INFO',
            'formatter': 'standard',
            'stream': 'ext://sys.stdout'
        },
        'file': {
            'class': 'logging.FileHandler',
            'level': 'DEBUG',
            'formatter': 'standard',
            'filename': 'debug.log',
            'mode': 'w',
        }
    },
    'loggers': {
        '': {  # root logger
            'handlers': ['console', 'file'],
            'level': 'DEBUG',
            'propagate': True
        }
    }
}

logging.config.dictConfig(config)
logger = logging.getLogger(__name__)

logger.debug("This goes to the file only")
logger.info("This goes to both console and file")
```

## Formatting Log Messages

The `format` parameter accepts a string with placeholders for various log record attributes:

| Format | Description |
|--------|-------------|
| %(asctime)s | Human-readable time when the LogRecord was created |
| %(created)f | Time when the LogRecord was created (as returned by time.time()) |
| %(filename)s | Filename portion of pathname |
| %(funcName)s | Name of function containing the logging call |
| %(levelname)s | Text logging level ('DEBUG', 'INFO', etc.) |
| %(levelno)d | Numeric logging level (10, 20, etc.) |
| %(lineno)d | Source line number where the logging call was issued |
| %(message)s | The logged message |
| %(module)s | Module from which logging call was made |
| %(name)s | Name of the logger |
| %(pathname)s | Full pathname of the source file |
| %(process)d | Process ID |
| %(processName)s | Process name |
| %(thread)d | Thread ID |
| %(threadName)s | Thread name |

Example:

```python
import logging

logging.basicConfig(
    level=logging.DEBUG,
    format='%(asctime)s [%(process)d] [%(threadName)s] [%(name)s] [%(levelname)s] - %(message)s - [%(filename)s:%(lineno)d]'
)

logging.debug("Detailed log message")
```

Output:
```
2025-05-04 12:34:56,789 [12345] [MainThread] [root] [DEBUG] - Detailed log message - [example.py:6]
```

## Logging to Files

### Basic File Logging

```python
import logging

logging.basicConfig(
    filename='app.log',
    filemode='a',  # 'w' to overwrite, 'a' to append
    level=logging.DEBUG,
    format='%(asctime)s - %(levelname)s - %(message)s'
)

logging.debug("This message goes to the log file")
```

### Rotating File Handler

Automatically rotates log files when they reach a certain size:

```python
import logging
from logging.handlers import RotatingFileHandler

logger = logging.getLogger(__name__)
logger.setLevel(logging.DEBUG)

# Create handlers
handler = RotatingFileHandler('app.log', maxBytes=10000, backupCount=5)
handler.setLevel(logging.DEBUG)

# Create formatters
formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
handler.setFormatter(formatter)

# Add handlers to logger
logger.addHandler(handler)

# Log messages
for i in range(100):
    logger.debug(f"This is log message {i}")
```

This will create files: app.log, app.log.1, app.log.2, etc., when the size exceeds 10KB.

### Time-Based Rotating File Handler

Rotates logs based on time intervals:

```python
import logging
from logging.handlers import TimedRotatingFileHandler

logger = logging.getLogger(__name__)
logger.setLevel(logging.DEBUG)

# Rotate at midnight each day, keep 7 backups
handler = TimedRotatingFileHandler(
    'app.log',
    when='midnight',
    interval=1,
    backupCount=7
)
handler.setLevel(logging.DEBUG)
handler.setFormatter(logging.Formatter('%(asctime)s - %(levelname)s - %(message)s'))

logger.addHandler(handler)

logger.info("This log will rotate at midnight")
```

## Multiple Handlers

You can direct logs to multiple destinations simultaneously:

```python
import logging
import sys

# Create logger
logger = logging.getLogger(__name__)
logger.setLevel(logging.DEBUG)

# Create console handler
console_handler = logging.StreamHandler(sys.stdout)
console_handler.setLevel(logging.WARNING)  # Only WARNING and above to console

# Create file handler
file_handler = logging.FileHandler('all_logs.log')
file_handler.setLevel(logging.DEBUG)  # All logs to file

# Create formatters
console_format = logging.Formatter('%(levelname)s: %(message)s')
file_format = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')

# Add formatters to handlers
console_handler.setFormatter(console_format)
file_handler.setFormatter(file_format)

# Add handlers to logger
logger.addHandler(console_handler)
logger.addHandler(file_handler)

# Log messages
logger.debug("This only goes to the file")
logger.warning("This goes to both console and file")
```

Output on console:
```
WARNING: This goes to both console and file
```

Output in file:
```
2025-05-04 12:34:56,789 - __main__ - DEBUG - This only goes to the file
2025-05-04 12:34:56,790 - __main__ - WARNING - This goes to both console and file
```

## Loggers and Logger Hierarchy

Loggers are organized in a hierarchy based on their names:

```python
import logging

# Configure root logger
logging.basicConfig(level=logging.WARNING)

# Create parent logger
parent_logger = logging.getLogger('parent')
parent_logger.setLevel(logging.INFO)

# Create child logger
child_logger = logging.getLogger('parent.child')

# Create another logger in different hierarchy
other_logger = logging.getLogger('other')

# Log messages
parent_logger.info("This is visible (INFO level)")
child_logger.info("This is also visible (inherits parent's level)")
other_logger.info("This is NOT visible (inherits root's WARNING level)")
```

Output:
```
INFO:parent:This is visible (INFO level)
INFO:parent.child:This is also visible (inherits parent's level)
```

### Propagation

By default, log messages propagate up the logger hierarchy:

```python
import logging

# Configure root logger with a handler
logging.basicConfig(
    level=logging.WARNING,
    format='ROOT: %(levelname)s - %(message)s'
)

# Create a custom logger with its own handler
logger = logging.getLogger('app')
logger.setLevel(logging.DEBUG)

handler = logging.StreamHandler()
handler.setFormatter(logging.Formatter('APP: %(levelname)s - %(message)s'))
logger.addHandler(handler)

# Log a message - it will appear twice!
logger.error("This is an error")
```

Output:
```
APP: ERROR - This is an error
ROOT: ERROR - This is an error
```

To prevent propagation:

```python
logger.propagate = False
logger.error("This appears only once")
```

Output:
```
APP: ERROR - This appears only once
```

## Filter Usage

Filters provide more fine-grained control over which log records get emitted:

```python
import logging

class SensitiveDataFilter(logging.Filter):
    def __init__(self, patterns=None):
        super().__init__()
        self.patterns = patterns or ['password', 'credit_card', 'ssn']
    
    def filter(self, record):
        # Check if any sensitive pattern exists in the log message
        message = record.getMessage().lower()
        for pattern in self.patterns:
            if pattern in message:
                # Redact the sensitive information
                record.msg = record.msg.replace(pattern, '***REDACTED***')
        return True

# Set up logging
logger = logging.getLogger(__name__)
logger.setLevel(logging.DEBUG)

handler = logging.StreamHandler()
handler.setLevel(logging.DEBUG)
formatter = logging.Formatter('%(asctime)s - %(levelname)s - %(message)s')
handler.setFormatter(formatter)

# Add the filter to the handler
data_filter = SensitiveDataFilter()
handler.addFilter(data_filter)

logger.addHandler(handler)

# Log with sensitive data
logger.info("User provided password: secret123")
logger.debug("Credit_card number entered: 1234-5678-9012-3456")
```

Output:
```
2025-05-04 12:34:56,789 - INFO - User provided ***REDACTED***: secret123
2025-05-04 12:34:56,790 - DEBUG - ***REDACTED*** number entered: 1234-5678-9012-3456
```

## Exception Information

Logging exceptions with traceback:

```python
import logging

logging.basicConfig(level=logging.DEBUG)

try:
    result = 10 / 0
except Exception as e:
    # Method 1: Using exc_info parameter
    logging.error("Division error occurred", exc_info=True)
    
    # Method 2: Using exception() method
    logging.exception("Alternative way to log exception")
    
    # Method 3: Manual formatting
    import traceback
    logging.error(f"Error details: {str(e)}\n{traceback.format_exc()}")
```

Output:
```
ERROR:root:Division error occurred
Traceback (most recent call last):
  File "example.py", line 6, in <module>
    result = 10 / 0
ZeroDivisionError: division by zero

ERROR:root:Alternative way to log exception
Traceback (most recent call last):
  File "example.py", line 6, in <module>
    result = 10 / 0
ZeroDivisionError: division by zero

ERROR:root:Error details: division by zero
Traceback (most recent call last):
  File "example.py", line 6, in <module>
    result = 10 / 0
ZeroDivisionError: division by zero
```

## Advanced Configuration

### Using YAML Configuration Files

Create a `logging_config.yaml` file:

```yaml
version: 1
formatters:
  simple:
    format: '%(asctime)s - %(name)s - %(levelname)s - %(message)s'
  detailed:
    format: '%(asctime)s - %(name)s - %(levelname)s - %(message)s - [%(filename)s:%(lineno)d]'
    datefmt: '%Y-%m-%d %H:%M:%S'
handlers:
  console:
    class: logging.StreamHandler
    level: INFO
    formatter: simple
    stream: ext://sys.stdout
  file:
    class: logging.FileHandler
    level: DEBUG
    formatter: detailed
    filename: app.log
    mode: a
  error_file:
    class: logging.FileHandler
    level: ERROR
    formatter: detailed
    filename: error.log
    mode: a
loggers:
  app:
    level: DEBUG
    handlers: [console, file]
    propagate: no
  app.models:
    level: DEBUG
    handlers: [file]
    propagate: yes
  app.api:
    level: INFO
    handlers: [console, file, error_file]
    propagate: no
root:
  level: WARNING
  handlers: [console]
```

Then load it in your Python code:

```python
import logging
import logging.config
import yaml

# Load the configuration
with open('logging_config.yaml', 'r') as f:
    config = yaml.safe_load(f)

logging.config.dictConfig(config)

# Get loggers
app_logger = logging.getLogger('app')
model_logger = logging.getLogger('app.models')
api_logger = logging.getLogger('app.api')

# Use the loggers
app_logger.debug("App debug message")
app_logger.info("App info message")
model_logger.debug("Model debug message")
api_logger.error("API error message")
```

### Using JSON Configuration

```python
import json
import logging.config

# Configuration as a Python dictionary
config = {
    "version": 1,
    "disable_existing_loggers": False,
    "formatters": {
        "standard": {
            "format": "%(asctime)s - %(name)s - %(levelname)s - %(message)s"
        }
    },
    "handlers": {
        "default": {
            "level": "INFO",
            "formatter": "standard",
            "class": "logging.StreamHandler"
        }
    },
    "loggers": {
        "": {
            "handlers": ["default"],
            "level": "INFO",
            "propagate": True
        }
    }
}

# You could also load from a file:
# with open('logging_config.json', 'r') as f:
#     config = json.load(f)

logging.config.dictConfig(config)
logger = logging.getLogger(__name__)
logger.info("Configuration loaded from JSON/dict")
```

### Using fileConfig with INI Files

Create a `logging.ini` file:

```ini
[loggers]
keys=root,simpleExample

[handlers]
keys=consoleHandler,fileHandler

[formatters]
keys=simpleFormatter

[logger_root]
level=DEBUG
handlers=consoleHandler

[logger_simpleExample]
level=DEBUG
handlers=fileHandler
qualname=simpleExample
propagate=0

[handler_consoleHandler]
class=StreamHandler
level=DEBUG
formatter=simpleFormatter
args=(sys.stdout,)

[handler_fileHandler]
class=FileHandler
level=DEBUG
formatter=simpleFormatter
args=('app.log', 'w')

[formatter_simpleFormatter]
format=%(asctime)s - %(name)s - %(levelname)s - %(message)s
datefmt=%Y-%m-%d %H:%M:%S
```

Then in your Python code:

```python
import logging
import logging.config

logging.config.fileConfig('logging.ini')

# Get the logger specified in the file
logger = logging.getLogger('simpleExample')
logger.debug('Debug message')
logger.info('Info message')
```

## Using ContextFilter for Additional Context

```python
import logging
import random
import threading

class RequestContextFilter(logging.Filter):
    """
    Add request-specific context to log records
    """
    def __init__(self):
        super().__init__()
        self.local = threading.local()
    
    def set_request_id(self, request_id):
        self.local.request_id = request_id
    
    def set_user_id(self, user_id):
        self.local.user_id = user_id
    
    def filter(self, record):
        # Add request_id attribute if available
        record.request_id = getattr(self.local, 'request_id', 'unknown')
        record.user_id = getattr(self.local, 'user_id', 'anonymous')
        return True

# Set up logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - [REQ:%(request_id)s] [USER:%(user_id)s] - %(levelname)s - %(message)s'
)

# Create and add the filter
context_filter = RequestContextFilter()
root_logger = logging.getLogger()
root_logger.addFilter(context_filter)

# Simulate request processing
def process_request(request_num):
    # Set context for this request
    request_id = f"{random.randint(1000, 9999)}"
    user_id = f"user_{random.randint(1, 100)}"
    
    context_filter.set_request_id(request_id)
    context_filter.set_user_id(user_id)
    
    # Log activities for this request
    logging.info(f"Processing request #{request_num}")
    logging.debug("Request details received")
    
    if random.random() < 0.3:
        logging.error("Error processing request")
    else:
        logging.info("Request processed successfully")

# Simulate multiple requests
for i in range(3):
    process_request(i+1)
```

Output:
```
2025-05-04 12:34:56,789 - [REQ:5678] [USER:user_42] - INFO - Processing request #1
2025-05-04 12:34:56,790 - [REQ:5678] [USER:user_42] - INFO - Request processed successfully
2025-05-04 12:34:56,791 - [REQ:3456] [USER:user_15] - INFO - Processing request #2
2025-05-04 12:34:56,792 - [REQ:3456] [USER:user_15] - ERROR - Error processing request
2025-05-04 12:34:56,793 - [REQ:7890] [USER:user_73] - INFO - Processing request #3
2025-05-04 12:34:56,794 - [REQ:7890] [USER:user_73] - INFO - Request processed successfully
```

## Best Practices

### 1. Use the Logger Hierarchy Effectively

Create loggers for different components:

```python
# In data_processor.py
import logging
logger = logging.getLogger(__name__)  # 'data_processor'

# In api_client.py
import logging
logger = logging.getLogger(__name__)  # 'api_client'

# In main.py - Configure all loggers
import logging

# Root configuration affects all loggers
logging.basicConfig(level=logging.WARNING)

# Configure specific modules
logging.getLogger('data_processor').setLevel(logging.DEBUG)
logging.getLogger('api_client').setLevel(logging.INFO)
```

### 2. Use Appropriate Log Levels

- **DEBUG**: Detailed diagnostic information
- **INFO**: Confirmation that things are working as expected
- **WARNING**: Something unexpected happened, but the application still works
- **ERROR**: An error occurred that prevented a function from working
- **CRITICAL**: A serious error that might prevent the program from continuing

### 3. Include Contextual Information

Always include relevant context in log messages:

```python
# Bad
logger.error("Database connection failed")

# Good
logger.error("Database connection failed: couldn't connect to %s:%d after %d retries over %d seconds",
             db_host, db_port, retries, timeout)
```

### 4. Use Structured Logging for Complex Applications

For applications that require advanced log analysis:

```python
import logging
import json

class JSONFormatter(logging.Formatter):
    """Format log records as JSON strings"""
    def format(self, record):
        # Create a dictionary with all the record attributes
        log_data = {
            'timestamp': self.formatTime(record),
            'level': record.levelname,
            'name': record.name,
            'message': record.getMessage(),
        }
        
        # Add exception info if present
        if record.exc_info:
            log_data['exception'] = self.formatException(record.exc_info)
        
        # Add any custom attributes
        for key, value in record.__dict__.items():
            if key not in ('args', 'asctime', 'created', 'exc_info', 'exc_text', 
                          'filename', 'funcName', 'id', 'levelname', 'levelno',
                          'lineno', 'module', 'msecs', 'message', 'msg', 'name', 
                          'pathname', 'process', 'processName', 'relativeCreated', 
                          'stack_info', 'thread', 'threadName') and not key.startswith('_'):
                log_data[key] = value
        
        return json.dumps(log_data)

# Set up logging with JSON formatter
logger = logging.getLogger(__name__)
handler = logging.StreamHandler()
handler.setFormatter(JSONFormatter())
logger.addHandler(handler)
logger.setLevel(logging.INFO)

# Log with extra context
logger.info("User logged in", extra={
    'user_id': '12345',
    'ip_address': '192.168.1.1',
    'session_id': 'abc123',
    'login_method': 'password'
})
```

Output:
```
{"timestamp": "2025-05-04 12:34:56,789", "level": "INFO", "name": "__main__", "message": "User logged in", "user_id": "12345", "ip_address": "192.168.1.1", "session_id": "abc123", "login_method": "password"}
```

### 5. Use a Complete Logging Example for a Production Application

```python
import logging
import logging.config
import os
import time
import json
from logging.handlers import RotatingFileHandler, SMTPHandler

def setup_logging(
    default_level=logging.INFO,
    env_key='LOG_CFG',
    log_dir='logs',
    app_name='myapp'
):
    """Setup logging configuration"""
    # Create log directory if it doesn't exist
    if not os.path.exists(log_dir):
        os.makedirs(log_dir)
    
    # Determine log file paths
    main_log = os.path.join(log_dir, f"{app_name}.log")
    error_log = os.path.join(log_dir, f"{app_name}_error.log")
    
    # Check for logging config file specified in env var
    path = os.getenv(env_key, None)
    if path and os.path.exists(path):
        with open(path, 'rt') as f:
            config = json.load(f)
        logging.config.dictConfig(config)
    else:
        # Create default config
        config = {
            'version': 1,
            'disable_existing_loggers': False,
            'formatters': {
                'standard': {
                    'format': '%(asctime)s [%(levelname)s] [%(name)s] %(message)s',
                    'datefmt': '%Y-%m-%d %H:%M:%S'
                },
                'detailed': {
                    'format': '%(asctime)s [%(levelname)s] [%(name)s] '
                              '[%(pathname)s:%(lineno)d] %(message)s',
                    'datefmt': '%Y-%m-%d %H:%M:%S'
                },
                'json': {
                    'format': '%(message)s',
                    '()': 'myapp.logging_utils.JSONFormatter'  # Custom formatter class
                }
            },
            'handlers': {
                'console': {
                    'class': 'logging.StreamHandler',
                    'level': 'INFO',
                    'formatter': 'standard',
                    'stream': 'ext://sys.stdout'
                },
                'file': {
                    'class': 'logging.handlers.RotatingFileHandler',
                    'level': 'DEBUG',
                    'formatter': 'detailed',
                    'filename': main_log,
                    'maxBytes': 10485760,  # 10MB
                    'backupCount': 10,
                    'encoding': 'utf8'
                },
                'error_file': {
                    'class': 'logging.handlers.RotatingFileHandler',
                    'level': 'ERROR',
                    'formatter': 'detailed',
                    'filename': error_log,
                    'maxBytes': 10485760,  # 10MB
                    'backupCount': 10,
                    'encoding': 'utf8'
                },
                'email': {
                    'class': 'logging.handlers.SMTPHandler',
                    'level': 'CRITICAL',
                    'formatter': 'detailed',
                    'mailhost': ('smtp.example.com', 587),
                    'fromaddr': 'alerts@example.com',
                    'toaddrs': ['admin@example.com'],
                    'subject': f"{app_name.upper()} CRITICAL ERROR ALERT",
                    'credentials': ('alerts@example.com', 'password'),
                    'secure': ()
                }
            },
            'loggers': {
                '': {  # root logger
                    'handlers': ['console', 'file', 'error_file', 'email'],
                    'level': default_level,
                    'propagate': True
                },
                'myapp': {
                    'handlers': ['console', 'file', 'error_file'],
                    'level': 'DEBUG',
                    'propagate': False
                },
                'myapp.api': {
                    'handlers': ['console', 'file', 'error_file'],
                    'level': 'INFO',
                    'propagate': False
                },
                'myapp.db': {
                    'handlers': ['file', 'error_file'],
                    'level': 'DEBUG',
                    'propagate': False
                }
            }
        }
        logging.config.dictConfig(config)


# Custom JSON formatter for structured logging
class JSONFormatter(logging.Formatter):
    def format(self, record):
        log_record = {
            "timestamp": self.formatTime(record, self.datefmt or "%Y-%m-%d %H:%M:%S"),
            "logger": record.name,
            "level": record.levelname,
            "thread": record.threadName,
            "process": record.processName,
            "message": record.getMessage(),
            "path": record.pathname,
            "line": record.lineno,
            "function": record.funcName
        }
        
        # Add exception info if available
        if record.exc_info:
            log_record["exception"] = self.formatException(record.exc_info)
        
        # Add any custom fields from the extra parameter
        for key, value in record.__dict__.items():
            if key not in log_record and not key.startswith('_') and key not in (
                'args', 'asctime', 'created', 'exc_info', 'exc_text', 'filename',
                'funcName', 'id', 'levelname', 'levelno', 'lineno', 'module',
                'msecs', 'msg', 'name', 'pathname', 'process', 'processName',
                'relativeCreated', 'stack_info', 'thread', 'threadName'
            ):
                log_record[key] = value
                
        return json.dumps(log_record)


# Usage example
if __name__ == "__main__":
    # Set up logging
    setup_logging(default_level=logging.DEBUG)
    
    # Get loggers
    root_logger = logging.getLogger()
    app_logger = logging.getLogger('myapp')
    api_logger = logging.getLogger('myapp.api')
    db_logger = logging.getLogger('myapp.db')
    
    # Log some messages
    root_logger.info("Application starting up")
    
    app_logger.debug("Initializing application components")
    api_logger.info("API service connecting to endpoints")
    db_logger.debug("Opening database connections")
    
    try:
        # Simulate an error
        result = 100 / 0
    except Exception as e:
        app_logger.error("Error during initialization", exc_info=True)
    
    app_logger.info("Application ready", extra={
        'startup_time': time.time(),
        'environment': os.environ.get('APP_ENV', 'development'),
        'version': '1.2.3'
    })
```

## Capturing Warnings

Python's warning system can be integrated with logging:

```python
import warnings
import logging

# Set up logging
logging.basicConfig(level=logging.INFO)

# Redirect warnings to the logging system
logging.captureWarnings(True)

# Create a warning
warnings.warn("This is a warning that will go to the logs")

# Issue a deprecated warning
warnings.warn("This feature will be removed in version 2.0", DeprecationWarning)
```

Output:
```
WARNING:py.warnings:UserWarning: This is a warning that will go to the logs
WARNING:py.warnings:DeprecationWarning: This feature will be removed in version 2.0
```

## Logging in Multithreaded Applications

Thread-safe logging with context:

```python
import logging
import threading
import random
import time

# Configure logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - [%(threadName)s] - %(levelname)s - %(message)s'
)
logger = logging.getLogger(__name__)

def worker(worker_id):
    """Worker function that simulates processing tasks"""
    logger.info(f"Worker {worker_id} starting")
    
    # Simulate work
    for i in range(3):
        sleep_time = random.uniform(0.5, 2.0)
        logger.debug(f"Worker {worker_id} processing task {i}")
        time.sleep(sleep_time)
        
        if random.random() < 0.2:  # 20% chance of error
            logger.error(f"Worker {worker_id} encountered an error on task {i}")
        else:
            logger.info(f"Worker {worker_id} completed task {i}")
    
    logger.info(f"Worker {worker_id} finished all tasks")

# Create and start threads
threads = []
for i in range(3):
    thread = threading.Thread(target=worker, args=(i,), name=f"Worker-{i}")
    threads.append(thread)
    thread.start()

# Wait for all threads to complete
for thread in threads:
    thread.join()

logger.info("All workers completed")
```

Output:
```
2025-05-04 12:34:56,789 - [Worker-0] - INFO - Worker 0 starting
2025-05-04 12:34:56,789 - [Worker-1] - INFO - Worker 1 starting
2025-05-04 12:34:56,790 - [Worker-2] - INFO - Worker 2 starting
2025-05-04 12:34:57,789 - [Worker-0] - INFO - Worker 0 completed task 0
2025-05-04 12:34:58,290 - [Worker-1] - INFO - Worker 1 completed task 0
2025-05-04 12:34:58,790 - [Worker-2] - ERROR - Worker 2 encountered an error on task 0
2025-05-04 12:34:59,789 - [Worker-0] - INFO - Worker 0 completed task 1
2025-05-04 12:35:00,290 - [Worker-1] - INFO - Worker 1 completed task 1
2025-05-04 12:35:01,290 - [Worker-2] - INFO - Worker 2 completed task 1
2025-05-04 12:35:01,789 - [Worker-0] - INFO - Worker 0 completed task 2
2025-05-04 12:35:01,789 - [Worker-0] - INFO - Worker 0 finished all tasks
2025-05-04 12:35:02,290 - [Worker-1] - ERROR - Worker 1 encountered an error on task 2
2025-05-04 12:35:02,290 - [Worker-1] - INFO - Worker 1 finished all tasks
2025-05-04 12:35:03,290 - [Worker-2] - INFO - Worker 2 completed task 2
2025-05-04 12:35:03,290 - [Worker-2] - INFO - Worker 2 finished all tasks
2025-05-04 12:35:03,291 - [MainThread] - INFO - All workers completed
```

## Queue Handler for Thread Safety

Using a QueueHandler for thread-safe logging:

```python
import logging
import random
import threading
import time
from logging.handlers import QueueHandler, QueueListener
import queue

# Create queue for logs
log_queue = queue.Queue()

# Configure handlers
console_handler = logging.StreamHandler()
console_handler.setLevel(logging.INFO)
console_handler.setFormatter(logging.Formatter('%(asctime)s - [%(threadName)s] - %(levelname)s - %(message)s'))

file_handler = logging.FileHandler('threaded_app.log')
file_handler.setLevel(logging.DEBUG)
file_handler.setFormatter(logging.Formatter('%(asctime)s - [%(threadName)s] - %(levelname)s - %(message)s'))

# Set up queue listener with handlers
listener = QueueListener(log_queue, console_handler, file_handler)
listener.start()

# Configure root logger with queue handler
root_logger = logging.getLogger()
root_logger.setLevel(logging.DEBUG)
root_logger.addHandler(QueueHandler(log_queue))

def worker(worker_id):
    """Worker function that logs messages"""
    logger = logging.getLogger(f"worker.{worker_id}")
    logger.info(f"Worker {worker_id} starting")
    
    # Generate log messages
    for i in range(5):
        time.sleep(random.uniform(0.1, 0.5))
        if random.random() < 0.2:
            logger.error(f"Error in task {i}")
        else:
            logger.debug(f"Processing task {i}")
            logger.info(f"Completed task {i}")
    
    logger.info(f"Worker {worker_id} finished")

# Create threads
threads = []
for i in range(3):
    t = threading.Thread(target=worker, args=(i,))
    threads.append(t)
    t.start()

# Wait for threads to finish
for t in threads:
    t.join()

logging.info("All workers have completed")

# Stop the listener
listener.stop()
```

## Using NullHandler

For library developers, it's best practice to add a NullHandler to avoid "No handler found" warnings:

```python
# In your library module:
import logging

# Create a logger for this module
logger = logging.getLogger(__name__)

# Add a null handler to avoid "No handler found" warnings
logger.addHandler(logging.NullHandler())

# Now you can use the logger in your library code
def some_function():
    logger.debug("Debug information")
    logger.info("Function called")
    
    try:
        # Some operation
        result = 1 / 0
    except Exception as e:
        logger.error("Error in function", exc_info=True)
```

## Customizing LogRecord Factory

You can customize how log records are created:

```python
import logging
import time
import threading

# Original LogRecord factory
original_factory = logging.getLogRecordFactory()

# Thread local storage for request context
thread_local = threading.local()

def set_request_context(request_id=None, user_id=None):
    """Set context for the current thread"""
    if request_id:
        thread_local.request_id = request_id
    if user_id:
        thread_local.user_id = user_id

def clear_request_context():
    """Clear the request context"""
    if hasattr(thread_local, 'request_id'):
        del thread_local.request_id
    if hasattr(thread_local, 'user_id'):
        del thread_local.user_id

# Custom LogRecord factory function
def request_context_record_factory(*args, **kwargs):
    # Create the log record using the original factory
    record = original_factory(*args, **kwargs)
    
    # Add request context
    record.request_id = getattr(thread_local, 'request_id', '-')
    record.user_id = getattr(thread_local, 'user_id', '-')
    
    return record

# Set the custom factory
logging.setLogRecordFactory(request_context_record_factory)

# Configure logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - [REQ:%(request_id)s] [USER:%(user_id)s] - %(levelname)s - %(message)s'
)

# Simulate request handling
def handle_request(request_number):
    # Set context for this request
    set_request_context(
        request_id=f"REQ-{request_number:04d}",
        user_id=f"user-{random.randint(1, 100)}"
    )
    
    logging.info(f"Processing request {request_number}")
    
    # Simulate processing
    time.sleep(random.uniform(0.1, 0.5))
    
    if random.random() < 0.3:
        logging.error("Error during request processing")
    else:
        logging.info("Request processed successfully")
    
    # Clear context when done
    clear_request_context()

# Process multiple requests
import random
for i in range(5):
    handle_request(i + 1)
```

Output:
```
2025-05-04 12:34:56,789 - [REQ:REQ-0001] [USER:user-42] - INFO - Processing request 1
2025-05-04 12:34:57,123 - [REQ:REQ-0001] [USER:user-42] - INFO - Request processed successfully
2025-05-04 12:34:57,456 - [REQ:REQ-0002] [USER:user-15] - INFO - Processing request 2
2025-05-04 12:34:57,789 - [REQ:REQ-0002] [USER:user-15] - ERROR - Error during request processing
2025-05-04 12:34:58,123 - [REQ:REQ-0003] [USER:user-78] - INFO - Processing request 3
2025-05-04 12:34:58,456 - [REQ:REQ-0003] [USER:user-78] - INFO - Request processed successfully
2025-05-04 12:34:58,789 - [REQ:REQ-0004] [USER:user-23] - INFO - Processing request 4
2025-05-04 12:34:59,123 - [REQ:REQ-0004] [USER:user-23] - INFO - Request processed successfully
2025-05-04 12:34:59,456 - [REQ:REQ-0005] [USER:user-91] - INFO - Processing request 5
2025-05-04 12:34:59,789 - [REQ:REQ-0005] [USER:user-91] - ERROR - Error during request processing
```

## Network-Based Logging

### Socket Handler

Send logs to a network socket:

```python
import logging
import logging.handlers

# Create logger
logger = logging.getLogger(__name__)
logger.setLevel(logging.DEBUG)

# Create socket handler
socketHandler = logging.handlers.SocketHandler(
    'localhost',
    logging.handlers.DEFAULT_TCP_LOGGING_PORT
)

# Add the handler to logger
logger.addHandler(socketHandler)

# Log messages
logger.debug("Debug message sent to socket")
logger.info("Info message sent to socket")
logger.warning("Warning message sent to socket")
```

### Syslog Handler

Send logs to a syslog server:

```python
import logging
from logging.handlers import SysLogHandler

# Create logger
logger = logging.getLogger(__name__)
logger.setLevel(logging.INFO)

# Create syslog handler
syslog = SysLogHandler(address=('localhost', 514))
formatter = logging.Formatter('%(name)s: %(levelname)s %(message)s')
syslog.setFormatter(formatter)

# Add the handler to logger
logger.addHandler(syslog)

# Log messages
logger.info("Info message sent to syslog")
logger.error("Error message sent to syslog")
```

### HTTP Handler

Send logs to an HTTP endpoint:

```python
import logging
import requests
from logging import Handler

class HTTPHandler(Handler):
    """Custom handler that sends logs to an HTTP endpoint"""
    
    def __init__(self, url, method='POST'):
        super().__init__()
        self.url = url
        self.method = method
    
    def emit(self, record):
        try:
            # Format the record
            log_entry = self.format(record)
            
            # Prepare payload
            payload = {
                'level': record.levelname,
                'message': record.getMessage(),
                'logger': record.name,
                'timestamp': self.formatter.formatTime(record),
                'log_entry': log_entry
            }
            
            # Add any exception info
            if record.exc_info:
                payload['exception'] = self.formatter.formatException(record.exc_info)
            
            # Send the request
            if self.method == 'POST':
                requests.post(self.url, json=payload, timeout=2.0)
            else:
                requests.get(self.url, params=payload, timeout=2.0)
                
        except Exception as e:
            # Don't raise exceptions from log handlers
            import sys
            print(f"Error sending log to HTTP endpoint: {e}", file=sys.stderr)

# Usage example
logger = logging.getLogger(__name__)
logger.setLevel(logging.INFO)

http_handler = HTTPHandler('https://logs.example.com/api/logs')
http_handler.setFormatter(logging.Formatter('%(asctime)s - %(levelname)s - %(message)s'))
logger.addHandler(http_handler)

logger.info("This log will be sent to an HTTP endpoint")
logger.error("Error message sent to HTTP endpoint", exc_info=True)
```

## Colorizing Log Output

For better console output readability:

```python
import logging
import sys

class ColorFormatter(logging.Formatter):
    """Formatter with colored output based on log level"""
    
    # ANSI color codes
    COLORS = {
        'DEBUG': '\033[94m',     # Blue
        'INFO': '\033[92m',      # Green
        'WARNING': '\033[93m',   # Yellow
        'ERROR': '\033[91m',     # Red
        'CRITICAL': '\033[91m\033[1m',  # Bold Red
        'RESET': '\033[0m'       # Reset
    }
    
    def format(self, record):
        # Get the original formatted message
        log_message = super().format(record)
        
        # Add color if the output is a terminal
        if sys.stdout.isatty():
            levelname = record.levelname
            if levelname in self.COLORS:
                log_message = f"{self.COLORS[levelname]}{log_message}{self.COLORS['RESET']}"
        
        return log_message

# Configure logging with color
logger = logging.getLogger(__name__)
logger.setLevel(logging.DEBUG)

# Create console handler
console = logging.StreamHandler()
console.setLevel(logging.DEBUG)

# Create formatter with color
formatter = ColorFormatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
console.setFormatter(formatter)

# Add handler to logger
logger.addHandler(console)

# Log messages with different levels
logger.debug("This is a DEBUG message")
logger.info("This is an INFO message")
logger.warning("This is a WARNING message")
logger.error("This is an ERROR message")
logger.critical("This is a CRITICAL message")
```

## Memory Handlers

Cache log records in memory and flush them based on specific triggers:

```python
import logging
from logging.handlers import MemoryHandler
import random

# Configure root logger
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

# Create a file handler that will receive flushed records
file_handler = logging.FileHandler('important_logs.log')
file_handler.setLevel(logging.DEBUG)
file_handler.setFormatter(logging.Formatter('%(asctime)s - %(levelname)s - %(message)s'))

# Create a memory handler that flushes to file_handler when an ERROR or higher is logged
# or when the buffer of 10 records is full
memory_handler = logging.handlers.MemoryHandler(
    capacity=10,                # Buffer up to 10 records
    flushLevel=logging.ERROR,   # Flush on ERROR or higher
    target=file_handler         # Flush to this handler
)
memory_handler.setLevel(logging.DEBUG)

# Add the memory handler to the logger
logger.addHandler(memory_handler)

# Generate some random logs
for i in range(20):
    level = random.choice([logging.DEBUG, logging.INFO, logging.WARNING, logging.ERROR])
    
    if level == logging.DEBUG:
        logger.debug(f"Debug message #{i}")
    elif level == logging.INFO:
        logger.info(f"Info message #{i}")
    elif level == logging.WARNING:
        logger.warning(f"Warning message #{i}")
    else:
        logger.error(f"Error message #{i}")

# Ensure all remaining records are flushed
memory_handler.flush()
```

This will write logs to 'important_logs.log' when either:
1. An ERROR message is logged
2. The buffer reaches 10 records

## Advanced Exception Handling

Capturing and logging detailed exception information:

```python
import logging
import traceback
import sys

# Configure logging
logging.basicConfig(
    level=logging.DEBUG,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
)
logger = logging.getLogger(__name__)

def log_detailed_exception(exc_type, exc_value, exc_traceback):
    """
    Log exception details including stack variables
    """
    # Don't log KeyboardInterrupt exceptions
    if issubclass(exc_type, KeyboardInterrupt):
        sys.__excepthook__(exc_type, exc_value, exc_traceback)
        return
        
    # Get the standard traceback info
    tb_lines = traceback.format_exception(exc_type, exc_value, exc_traceback)
    tb_text = ''.join(tb_lines)
    
    # Add the traceback to the log
    logger.error(f"Uncaught exception:\n{tb_text}")
    
    # Log local variables for each frame
    logger.error("Local variables by frame:")
    
    current_tb = exc_traceback
    while current_tb:
        frame = current_tb.tb_frame
        logger.error(f"Frame {frame.f_code.co_name} in {frame.f_code.co_filename} at line {frame.f_lineno}")
        
        for key, value in frame.f_locals.items():
            # Limit the length of the logged values
            value_str = str(value)
            if len(value_str) > 500:
                value_str = value_str[:500] + "..."
                
            logger.error(f"    {key} = {value_str}")
            
        current_tb = current_tb.tb_next
        
# Replace the default exception handler
sys.excepthook = log_detailed_exception

# Example function that will raise an exception
def divide_numbers(a, b):
    result = a / b
    return result

def process_calculation():
    values = [10, 20, 0, 30]
    results = []
    
    for i, val in enumerate(values):
        try:
            result = divide_numbers(100, val)
            results.append(result)
            logger.info(f"Calculation {i} succeeded: 100/{val} = {result}")
        except Exception as e:
            logger.error(f"Calculation {i} failed with {type(e).__name__}: {str(e)}")
            # Re-raise to demonstrate the excepthook
            raise

# Run the function that will raise an exception
try:
    process_calculation()
except:
    pass  # We're just demonstrating the logging, not handling the exception here
```

## Performance Considerations

### Lazy Evaluation with % Formatting

```python
import logging
import time

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

def expensive_operation():
    """Simulate an expensive operation that produces a string"""
    time.sleep(1)  # Simulate expensive work
    return "result of expensive operation"

# Good - lazy evaluation, only executes if the log level is enabled
logger.debug("Debug message with %s", expensive_operation())  

# Bad - always executes expensive_operation() even if debug is disabled
logger.debug("Debug message with " + expensive_operation())
```

### Using isEnabledFor for Complex Logging

```python
import logging
import time
import json

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

def generate_complex_debug_data():
    """Generate a complex data structure for debugging - expensive operation"""
    time.sleep(0.5)  # Simulate expensive work
    data = {
        "users": [{"id": i, "name": f"User {i}", "status": "active"} for i in range(100)],
        "transactions": [{"id": i, "amount": i * 10.5, "status": "completed"} for i in range(50)],
        "system_metrics": {
            "cpu": 45.2,
            "memory": 60.8,
            "disk": 78.3,
            "network": [{"interface": "eth0", "rx": 10240, "tx": 20480}]
        }
    }
    return data

# Efficient approach - check level first
if logger.isEnabledFor(logging.DEBUG):
    debug_data = generate_complex_debug_data()
    logger.debug("System state: %s", json.dumps(debug_data, indent=2))
```

## Monitoring and Auditing

### Using LoggerAdapter for Context

```python
import logging
import uuid
import random
import time

# Basic configuration
logging.basicConfig(level=logging.INFO, 
                    format='%(asctime)s - %(levelname)s - %(message)s')

class RequestAdapter(logging.LoggerAdapter):
    """Adapter that adds request context to log records"""
    
    def process(self, msg, kwargs):
        # Add request_id to the message
        return f'[Request ID: {self.extra["request_id"]}] {msg}', kwargs

# Simulate a web server that processes requests
def process_request(request_id, path, method):
    # Create a base logger
    base_logger = logging.getLogger("webserver")
    
    # Create an adapter with request context
    logger = RequestAdapter(base_logger, {"request_id": request_id})
    
    # Log the request
    logger.info(f"Received {method} request for {path}")
    
    # Simulate processing
    processing_time = random.uniform(0.1, 0.5)
    time.sleep(processing_time)
    
    # Sometimes generate errors
    if random.random() < 0.3:
        logger.error(f"Error processing {method} request for {path}")
        return 500
    
    # Log completion
    logger.info(f"Completed {method} request for {path} in {processing_time:.2f}s")
    return 200

# Simulate some HTTP requests
paths = ["/api/users", "/api/products", "/api/orders", "/login", "/logout"]
methods = ["GET", "POST", "PUT", "DELETE"]

for _ in range(5):
    request_id = str(uuid.uuid4())
    path = random.choice(paths)
    method = random.choice(methods)
    status = process_request(request_id, path, method)
```

Output:
```
2025-05-04 12:34:56,789 - INFO - [Request ID: 123e4567-e89b-12d3-a456-426614174000] Received GET request for /api/users
2025-05-04 12:34:57,123 - INFO - [Request ID: 123e4567-e89b-12d3-a456-426614174000] Completed GET request for /api/users in 0.33s
2025-05-04 12:34:57,124 - INFO - [Request ID: 234e5678-e89b-12d3-a456-426614174000] Received POST request for /login
2025-05-04 12:34:57,456 - ERROR - [Request ID: 234e5678-e89b-12d3-a456-426614174000] Error processing POST request for /login
2025-05-04 12:34:57,457 - INFO - [Request ID: 345e6789-e89b-12d3-a456-426614174000] Received PUT request for /api/products
2025-05-04 12:34:57,789 - INFO - [Request ID: 345e6789-e89b-12d3-a456-426614174000] Completed PUT request for /api/products in 0.33s
```

## Conclusion

Python's logging module is a powerful and flexible system that allows for sophisticated log management from simple console output to distributed logging across multiple applications.

Key takeaways:
1. Use appropriate log levels for different types of information
2. Configure loggers, handlers, and formatters for fine-grained control
3. Take advantage of logger hierarchies for component-specific logging
4. Add context to logs for better troubleshooting
5. Consider performance implications, especially for debug logging
6. Use structured logging for easier analysis in large applications

By implementing these practices, you can create a robust logging system that supports both development and production environments effectively.
