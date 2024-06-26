# Description
Setting up a logger for the PySB library.

# Code
```
import logging
import platform
import socket
import time
import os
import warnings
import pysb

SECONDS_IN_HOUR = 3600
LOG_LEVEL_ENV_VAR = 'PYSB_LOG'
BASE_LOGGER_NAME = 'pysb'
EXTENDED_DEBUG = 5
NAMED_LOG_LEVELS = {'NOTSET': logging.NOTSET,
                    'EXTENDED_DEBUG': EXTENDED_DEBUG,
                    'DEBUG': logging.DEBUG,
                    'INFO': logging.INFO,
                    'WARNING': logging.WARNING,
                    'ERROR': logging.ERROR,
                    'CRITICAL': logging.CRITICAL}

def formatter(time_utc=False):
    log_fmt = logging.Formatter('%(asctime)s.%(msecs).3d - %(name)s - '
                                '%(levelname)s - %(message)s',
                                datefmt='%Y-%m-%d %H:%M:%S')
    if time_utc:
        log_fmt.converter = time.gmtime

def setup_logger(level=logging.WARNING, console_output=True, file_output=False,
                 time_utc=False, capture_warnings=True):
    """
    Set up a new logging.Logger for PySB logging

    Calling this method will override any existing handlers, formatters etc.
    attached to the PySB logger. Typically, :func:`get_logger` should be
    used instead, which returns the existing PySB logger if it has already
    been set up, and can handle PySB submodule namespaces.

    Parameters
    ----------
    level : int
        Logging level, typically using a constant like logging.INFO or
        logging.DEBUG
    console_output : bool
        Set up a default console log handler if True (default)
    file_output : string
        Supply a filename to copy all log output to that file, or set to
        False to disable (default)
    time_utc : bool
        Specifies whether log entry time stamps should be in local time
        (when False) or UTC (True). See :func:`formatter` for more
        formatting options.
    capture_warnings : bool
        Capture warnings from Python's warnings module if True (default)

    Returns
    -------
    A logging.Logger object for PySB logging. Note that other PySB modules
    should use a logger specific to their namespace instead by calling
    :func:`get_logger`.
    """
    log = logging.getLogger(BASE_LOGGER_NAME)

    # Logging level can be overridden with environment variable
    if LOG_LEVEL_ENV_VAR in os.environ:
        try:
            level = int(os.environ[LOG_LEVEL_ENV_VAR])
        except ValueError:
            # Try parsing as a name
            level_name = os.environ[LOG_LEVEL_ENV_VAR]
            if level_name in NAMED_LOG_LEVELS.keys():
                level = NAMED_LOG_LEVELS[level_name]
            else:
                raise ValueError('Environment variable {} contains an '
                                 'invalid value "{}". If set, its value must '
                                 'be one of {} (case-sensitive) or an '
                                 'integer log level.'.format(
                    LOG_LEVEL_ENV_VAR, level_name,
                    ", ".join(NAMED_LOG_LEVELS.keys())
                                 ))

    log.setLevel(level)

    # Remove default logging handler
    log.handlers = []

    log_fmt = formatter(time_utc=time_utc)

    if console_output:
        stream_handler = logging.StreamHandler()
        stream_handler.setFormatter(log_fmt)
        log.addHandler(stream_handler)

    if file_output:
        file_handler = logging.FileHandler(file_output)
        file_handler.setFormatter(log_fmt)
        log.addHandler(file_handler)

    log.info('Logging started on PySB version %s', pysb.__version__)
    if time_utc:
        log.info('Log entry times are in UTC')
    else:
        utc_offset = time.timezone if (time.localtime().tm_isdst == 0) else \
            time.altzone
        utc_offset = -(utc_offset / SECONDS_IN_HOUR)
        log.info('Log entry time offset from UTC: %.2f hours', utc_offset)

    log.debug('OS Platform: %s', platform.platform())
    log.debug('Python version: %s', platform.python_version())
    log.debug('Hostname: %s', socket.getfqdn())

    logging.captureWarnings(capture_warnings)


```
