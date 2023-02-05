# PHP-FPM tasks

## Source issues

### FastCGI, worker request handling

- **Bug**: main - correctly decode path_info before comparing to comply with cgi spec
  - https://bugs.php.net/bug.php?id=74129 - Incorrect SCRIPT_NAME with apache ProxyPassMatch when spaces are in path
- **Bug**: main - verify the PATH_TRANSLATED logic with cgi.fix_pathinfo
  - https://bugs.php.net/bug.php?id=68053 - PHP-FPM with cgi.fix_pathinfo 0 loads script from PATH_TRANSLATED
- **Bug**: main - Look to the suggested changes for Apache and fcgi env vars (mainly verify if it's still issue with httpd)
  - https://bugs.php.net/bug.php?id=51983 - [fpm sapi] pm.status_path not working when cgi.fix_pathinfo=1
- **Test**: main - properly test the logic for processing env vars (especially the httpd logic) and verify it works with httpd balancer
  - https://bugs.php.net/bug.php?id=71379 - Add support for Apache 2.4 mod_proxy_balancer to FPM
  - https://bugs.php.net/bug.php?id=67998 - Wrong SCRIPT_FILENAME (check if this is still an issue or has been fixed)
- **Bug**: main - argv and argc should not be included in env vars
  - https://bugs.php.net/bug.php?id=75712 - php-fpm's import_environment_variables impl should not copy $_ENV, $_SERVER
- **Bug**: main - check handling of multiple headers
  - https://bugs.php.net/bug.php?id=78844 - FPM does not support multiple HTTP request headers with the same name
- **Bug**: fcgi - Abort connection if CONTENT_LENGTH differs from input length
- **Bug**: fcgi - FCGI_GET_VALUES does not seem to work
  - https://bugs.php.net/bug.php?id=76922 - FastCGI terminates connection immediately after FCGI_GET_VALUES
- **Bug**: fcgi - FCGI_ABORT_REQUEST from the client (web server) seems to be ignored
  - https://bugs.php.net/bug.php?id=76419 - connection_aborted() under FPM doesn't work properly
- **Bug**: fcgi / main - Check logic around flushing after using fastcgi_finish_request()
  - https://github.com/php/php-src/issues/9741 - flush() halts script after fastcgi_finish_request()
- **Bug**: fcgi - Investigate deadlock for keepalive connection after fastcgi_finish_request()
  - https://github.com/php/php-src/issues/10335 - FPM: keepalived connection with fastcgi_finish_request causes dead lock
- **Feat**: fcgi / main - look to support for close parameter in fastcgi_finish_request
  - https://github.com/php/php-src/pull/10273 - FPM: fastcgi_finish_request supports force close connection param
- **Feat**: fcgi - flag to allow missing content length
  - https://github.com/php/php-src/pull/7509 - discussion
- **Feat**: fcgi / main - refactore processing of fcgi env vars
- **Feat**: fcgi - spec update to no content length bytes
  - https://bugs.php.net/bug.php?id=79723 - sapi_cgi_read_post() ignores EOF
  - https://bugs.php.net/bug.php?id=51191 - Request body is 0-size when chunked requests are used 
  - https://github.com/php/php-src/pull/7509 - Fixed reading in streamed body using fastcgi
- **Feat**: fcgi - do not require any CGI parameters to allow overwriting on pool level
  - https://bugs.php.net/bug.php?id=74995 - Control/set fastcgi parameters from pool config file
- **Feat**: fcgi - spec update to allow using FCGI over TLS
- **Feat**: fcgi - spec update to allow larger headers than 64k
  - https://trac.nginx.org/nginx/ticket/239 - Support for large (> 64k) FastCGI requests
- **Feat**: fcgi - implement TLS support with client cert auth
- **Feat**: general - review return values in FPM code
  - https://github.com/php/php-src/pull/9911 - Change return values to bool or void in FPM
- **Feat**: general - Convert worker pools and children list to continous array for fast look up in stdio 

### Logging, tracing and socket

- **Bug**: error log - check how changes is error log file permission impacts IP and time logged
  - https://bugs.php.net/bug.php?id=62660 - PHP error logging with FPM fails to display an IP address and correct time
- **Bug**: error log - Investigate why error goes to stderr instead of error log file
  - https://bugs.php.net/bug.php?id=63555 - errors outputed to stderr instead of logfile using fastcgi
- **Bug**: stdio - Nodaemonized FPM in Bash background process hangs
  - https://github.com/php/php-src/issues/10058 - If "php-fpm --nodaemonize" is called to be sent into background it hangs the calling process 
- **Bug**: access log - fpm_log_format needs cleanup
  - https://bugs.php.net/bug.php?id=75635 - Memory leak in fpm log
- **Bug**: access log - %t shows the first request and not the last request
  - https://bugs.php.net/bug.php?id=69367 - access.log invalid time request
- **Feat**: access log - respect locale for time - consider logging afte after request shutdown
  - https://bugs.php.net/bug.php?id=69561 - FPM access.log strftime locale not configurable
- **Feat**: access log - add support stime and utime options
  - https://bugs.php.net/bug.php?id=62951 - Log utime and stime (patch)
- **Feat**: access log - fmt flag for path info or available env to use
  - https://bugs.php.net/bug.php?id=81670 - Access log contains wrong values for "%r" (request URI) format string
- **Feat**: access log - Allow suppression of queries
  - https://github.com/php/php-src/issues/10116 - /status?json&full is not suppressed when access.suppress_path used
- **Feat**: access log - long lines support - using the same logic as zlog ideally
  - https://github.com/php/php-src/pull/5634 - PR to discussiong it and removing unused MAX_LINE_LENGTH
- **Feat**: log - allow separation access and error log to stdout and stderr
  - https://bugs.php.net/bug.php?id=73886 - Handle access log & error log on Docker
  - https://github.com/php/php-src/pull/2310 - Fix #73886: Handle access log & error log on Docker (discussion)
- **Feat**: error log - master logging should be separated from child logs (special option for master log)
  - https://bugs.php.net/bug.php?id=69662 - PHP Startup errors are erroneously logged as master user in pool error log
  - https://bugs.php.net/bug.php?id=72357 - Pool logs created with master owner:group
- **Feat**: trace - slowlog - the SIGSTOP and SIGCONT stop script connection in stream (fsockopen) or mysql
  - https://bugs.php.net/bug.php?id=67471 - fpm slow log && fsockopen :Operation now in progress
  - https://bugs.php.net/bug.php?id=67087 - slowlog kills mysql connection
- **Feat**: trace - slowlog - option to set file permission for slow log and possibly other log files
  - https://bugs.php.net/bug.php?id=69509 - fpm slowlog file permissions feature
  - https://bugs.php.net/bug.php?id=61435 - PHP-FPM logs are not readable by group/others by default
  - https://github.com/php/php-src/pull/771 - fpm: relax log permissions (bug #61435)
- **Feat**: trace - slowlog - add request information - maybe some custom configurable fmt
  - https://bugs.php.net/bug.php?id=81501 - Log request information in slowlog
  - https://bugs.php.net/bug.php?id=79137 - Add request parameters to slow log script report
- **Bug**: socket - Add FD_CLOSEXEC on listening socket so it is not iherited by proc_open
  - https://bugs.php.net/bug.php?id=67383 - exec() leaks file and socket descriptors to called program (general for all parts of PHP - not good idea)
  - https://bugs.php.net/bug.php?id=76067 - system() function call leaks php-fpm listening sockets
  - https://bugs.php.net/bug.php?id=80992 - fork() don't close fpm_globals.listening_socket
- **Bug**: socket - non portable socket binding on FreeBSD
  - https://bugs.php.net/bug.php?id=77501 - listen = 9000 only listens on one interface
  - https://bugs.php.net/bug.php?id=77482 - Wont bind to IPv4 if IPv6 enabled
- **Feat**: socket - socket route option
  - https://github.com/php/php-src/pull/8470 - FPM add routing view global option (for FreeBSD for now).
- **Feat**: socket - improve listen queue status info
  - https://github.com/php/php-src/issues/9943 - FPM improve listen queue status info
  - https://bugs.php.net/bug.php?id=76323 - FPM /status reports wrong number of listen queue len
  - https://bugs.php.net/bug.php?id=80739 - PHP-FPM status page shows listen queue 0 (main details here)
- **Feat**: socket - Consider re-enabling warning for non empty listening queue or remove commented out code in process ctl.
  - https://github.com/php/php-src/blob/2ac5948f5cbaf3351fe18ab1068422487c9c215f/sapi/fpm/fpm/fpm_process_ctl.c#L349-L359

### Status and config

- **Bug**: status - JSON and XML output is not escaped and can be invalid
  - https://bugs.php.net/bug.php?id=69250 - PHP FPM status report produces invalid JSON and XML
  - https://bugs.php.net/bug.php?id=64539 - FPM status page: query_string not properly JSON encoded
- **Bug**: status - start time invalid on Solaris
  - https://bugs.php.net/bug.php?id=69289 - fpm_status uses wrong time_format for json and xml output
- **Feat**: status - support full parameter for openmetrics
  - https://github.com/php/php-src/issues/9494 - FPM status with OpenMetrics format and FULL parameter
  - https://github.com/php/php-src/pull/9646 - expose per process metrics in fpm status
- **Feat**: status - value queries and v2 openmetrics
  - https://github.com/php/php-src/pull/7291#discussion_r676184017
- **Feat**: status - add parameters to list extra fields for fcgi envs
  - https://github.com/php/php-src/issues/8880 - Support fastcgi parameter in FPM for status page to display request meta data
  - https://github.com/php/php-src/pull/2713 - Enhancement: php-fpm status page shows ip address of the client (closed but idea is there)
  - https://bugs.php.net/bug.php?id=72319 - FPM status page shows wrong request_uri (generic solution for this bug - would be worth to do path info as well)
- **Feat**: status / ping - healtcheck support based on metrics similar to https://github.com/renatomefi/php-fpm-healthcheck
- **Feat**: ping - integrate ping.listen similar to pm.status_listen
  - https://bugs.php.net/bug.php?id=68678 - FPM Ping should use a reserved worker
- **Feat**: scoreboard - track number of terminated requests
  - https://bugs.php.net/bug.php?id=78789 - Count number of request_terminate_timeout reached + killed in scoreboard
- **Bug**: conf - possibly incorrect order of ini setting
  - https://bugs.php.net/bug.php?id=75741 - enable_post_data_reading not working on PHP-FPM
  - https://github.com/php/php-src/issues/8157 - post_max_size evaluates .user.ini too late in php-fpm?
  - https://github.com/php/php-src/pull/8955 - activate sapi module before POST handling to allow to apply user.ini
- **Bug**: conf - possible overwritting issue with php_admin_value and php_value
  - https://bugs.php.net/bug.php?id=80012 - PHP_ADMIN_VALUE parameter is not correctly inherited
  - https://bugs.php.net/bug.php?id=72253 - phpinfo shows only first block of admin_value[disable_functions]
  - https://bugs.php.net/bug.php?id=72018 - php_admin_value override
  - https://bugs.php.net/bug.php?id=68171 - php_admin_value[extension] order not maintained
  - https://bugs.php.net/bug.php?id=68018 - php_value directive modifies "Changeable" context (patch)
  - https://bugs.php.net/bug.php?id=60387 - Problem with php_(admin)?_value/flag and load order
  - https://github.com/php/php-src/issues/8398 - php_value[xxx] in php-fpm pool - first declaration wins
- **Bug**: conf - investigate issue with loading some extensions in pool config
  - https://github.com/php/php-src/issues/9921 - php_admin_value[extension] = ext.so in fpm config does not register module handlers
- **Feat**: conf - look to supporting zend_extension in php_admin_value
  - https://bugs.php.net/bug.php?id=73408 - Loading Zend Extensions in FPM Pool Configuration
- **Feat**: conf - Introduce ZEND_INI_ADMIN for PHP ini admin values and not overwrite some system ini that cannot be overriden
  - https://github.com/php/php-src/issues/8699 - Wrong value from ini_get() for shared files because of opcache optimization
  - https://github.com/php/php-src/issues/9722 - FPM is maliciously compliant when asked to set opcache.preload
  - https://github.com/php/php-src/issues/10117 - php_admin_[flag|value] falsely applies config changes
  - https://github.com/php/php-src/pull/8997 - Fix GH-8699: ini_get() opcache optimization in 8.1 (see discussion of the change)
- **Feat**: main - PHP_ADMIN_VALUE and PHP_VALUE should be cleared on request end
  - https://bugs.php.net/bug.php?id=63965 - php-fpm site-specific settings go global
  - https://bugs.php.net/bug.php?id=53611 - fastcgi_param PHP_VALUE pollutes other sites
  - https://bugs.php.net/bug.php?id=61867 - user_ini_filename ignored for cached user_config (might need a check if this will be addressed by the above)
- **Feat**: main - option disable PHP_ADMIN_VALUE and PHP_VALUE
  - https://bugs.php.net/bug.php?id=64103 - Bypass PHP Security Settings with PHP-FPM
  - https://bugs.php.net/bug.php?id=70134 - open_basedir bypass with IP-based PHP-FPM
  - https://bugs.php.net/bug.php?id=72129 - PHP_VALUE, PHP_ADMIN_VALUE... changed by environment variables set in .htaccess
  - https://bugs.php.net/bug.php?id=77190 - PHP_VALUE can be changed by PHP-FPM environment variables without checking
  - https://bugs.php.net/bug.php?id=78305 -	Injection of INI settings into FPM worker processes
  - https://bugs.php.net/bug.php?id=79417 - PHP-FPM: php_admin_value can be overwritten in .htaccess
  - https://bugs.php.net/bug.php?id=80385 - Response data preceded by post data
- **Bug**: conf - setting env[LC_MESSAGES] does not work as expected
  - https://bugs.php.net/bug.php?id=81253 - unexpected result of setting env[LC_MESSAGES] in php-fpm pool config
- **Feat**: conf - add support for environment variables in configuration
  - https://bugs.php.net/bug.php?id=75994 - Environment permanently breaks for worker process.
  - https://bugs.php.net/bug.php?id=76798 - Can't configure PHP-FPM via environment variables (check if it worked before 7.1.15 and 7.2.1)
- **Feat**: conf - allow multiple includes per file
  - https://bugs.php.net/bug.php?id=68022 -	FPM should allow multiple includes per file (patch)
- **Bug**: conf - if error in the config, correctly put file name and file line of the error to the error message
  - https://bugs.php.net/bug.php?id=65500 - debug_backtrace doesn't identify file name when config contains invalid comment
- **Feat**: conf - optional pool prefix support
  - https://github.com/php/php-src/pull/3563 - Introducing "pool:" prefix on [section] FPM ini configuration
- **Feat**: conf - add compile option to set default FPM config path
  - https://bugs.php.net/bug.php?id=61073 - php-fpm config file path option lacking
- **Feat**: conf - Review FPM_PHP_INI_TO_EXPAND list in fpm_php.h as some are outdated

### Process management and related

- **Bug**: core - crash with Runkit extension - dangling pointer
  - https://bugs.php.net/bug.php?id=72055 - php-fpm crashes on working with Runkit (contains patch)
- **Bug**: core - huge pages enabled crash (might be opcache)
  - https://bugs.php.net/bug.php?id=81444 - php-fpm crashes with bus error under kubernetes
- **Bug**: core - opcache doesn't work with fpm chroot
  - https://bugs.php.net/bug.php?id=67141 - PHP FPM vhost pollution
- **Bug**: unix - validate extension_dir setting with chroot set
  - https://bugs.php.net/bug.php?id=64151 - Invalid include extension_dir
- **Bug**: unix - it should allow rewriting fcgi path envs if chroot enabled (should be probably optional)
  - https://bugs.php.net/bug.php?id=62279 - PHP-FPM chroot never-solved problems (extends #55322)
  - https://bugs.php.net/bug.php?id=55322 - Apache : TRANSLATED_PATH doesn't consider chroot
- **Bug**: systemd - check how PrivateTmp should work with chroot
  - https://bugs.php.net/bug.php?id=73466 - systemd option PrivateTmp= having no effect for a pool that is chrooted.
- **Bug**: systemd - check strange output of the fpm process line in systemd (only some version of it)
  - https://github.com/php/php-src/issues/10204 - Weird systemctl status phpX.Y-fpm output on Ubuntu 22.04
- **Feat**: systemd - Add CapabilityBoundingSet
  - https://github.com/php/php-src/pull/4960 - Add CapabilityBoundingSet to systemd unit file (my PR)
- **Bug**: signal - sigpipe might cause fpm to be unresponsive
  - https://bugs.php.net/bug.php?id=67320 - Ignored sigpipes in php-fpm cause php to become unresponsive
- **Feat**: signal: consider changing SIGTERM for primary idle killing instead of SIGQUIT
  - https://github.com/php/php-src/blob/8f02d7b7e499490312969cdfa9a3a81b9458595a/sapi/fpm/fpm/fpm_process_ctl.c#L138-L139
- **Bug**: event - check for the maximum file descriptors in devpoll
  - https://bugs.php.net/bug.php?id=65774 - no max file descriptor check for events.mechanism = /dev/poll
- **Bug**: event - FreeBSD kqueue time outs
  - https://bugs.php.net/bug.php?id=76630 - php-fpm time outs unexpectedly
- **Bug**: proc - process_control_timeout wakes up script in sleep
  - https://bugs.php.net/bug.php?id=77603 - Unexpected behavior with reloading php-fpm and sleep method
- **Bug**: proc - ondemand race condition
  - https://github.com/php/php-src/pull/1308 - pm.ondemand forks fewer child workers than it should
  - https://bugs.php.net/bug.php?id=69724 - pm.ondemand forks fewer child workers than it should (bug for the above PR - contains extra patches)
  - https://bugs.php.net/bug.php?id=72935 - ONDEMAND: Race condition causes incoming connections hang
- **Feat**: proc - alternative for process idle for better scaling in ondemend mode - might need some refactoring:
  - https://bugs.php.net/bug.php?id=78405 - Fpm keeps killing idle children claiming they timed out
  - https://bugs.php.net/bug.php?id=77060 - PHP-FPM pm.process_idle_timeout behaviour (doc bug confusion asking about this behaviour)
- **Feat**: proc - look to some option in ondemand for killing inactive connections
  - https://bugs.php.net/bug.php?id=69890 - pm.ondemand does not kill children after reaching max limit
- **Feat**: proc - stage redefinition to consider children idle if in reading header stage
  - https://bugs.php.net/bug.php?id=78405 - FPM with keepalive: Kills worker when no request is seen for $terminate_timeout
  - https://github.com/php/php-src/pull/8163 - Fix #78405: FPM with keepalive kills workers after $terminate_timeout
- **Feat**: proc - look to ondemand scheduling issues / improvements - using epoll (optionally for all modes)
  - https://bugs.php.net/bug.php?id=77959 - Scheduling of PHP-FPM processes in "ondemand"
  - https://github.com/php/php-src/pull/4101 - epoll discussion
  - https://github.com/php/php-src/pull/4104 - correct computation of idle time
  - https://bugs.php.net/bug.php?id=68824 - php-rpm pm=static causes load peaks when pm.max_requests is reached (epoll should resolve this)
  - https://bugs.php.net/bug.php?id=77060 - PHP-FPM pm.process_idle_timeout behaviour
- **Feat**: proc - look to using reuse port for listen based scheduling (round robin) instead of fcgi lock
- **Feat**: proc - Redefine boundary of pm.start_servers and pm.min_spare_servers
  - https://bugs.php.net/bug.php?id=62630 - issues with starting FPM in conditions of high load
- **Feat**: proc - consider defining timeout when no children available
  - https://bugs.php.net/bug.php?id=65503 - Timeout when max_children reached
- **Feat**: proc and main - Bootstrapping mode
  - https://github.com/php/php-src/pull/6772 - Add FPM early bootstrapping mode
- **Feat**: proc - Introduce delay for process restarts to prevent CPU exhaustion
  - https://github.com/php/php-src/issues/9632 - FPM delayed process restarting
- **Feat**: proc - add function to terminate child (this should be probably explicitly enabled in pool config)
  - https://bugs.php.net/bug.php?id=62948 - apache_child_terminate() for FPM
- **Feat**: event - make default timeout in event loop configurable (currently hard coded 1s)
  - https://bugs.php.net/bug.php?id=71854 - FPM: Please make epoll sleep interval configurable
- **Feat**: event - Cleaning up child events when the child is killed
  - https://github.com/php/php-src/pull/9817 - FPM delete child log event before child is killed
- **Feat**: proc - remove extra zeros in proc name
  - https://bugs.php.net/bug.php?id=77044 - Process Name has lots of null appended
- **Feat**: unix - extra check for selinux deny_ptrace
  - https://github.com/php/php-src/pull/7648 - fpm dumpable process setting extra check for SElinux based systems.
- **Feat**: unix - middle ground between chrooted and non-chrooted env (ideas from suphp)
  - https://bugs.php.net/bug.php?id=68125 - FPM check if script is in specified path before execute (ie docroot)
- **Feat**: reload - consider using SIGHUP as another reload signal
  - https://bugs.php.net/bug.php?id=67553 - Add SIGHUP as a reload signal
- **Feat**: reload - reduce number of reallocation or use buffering (stack or smart str) for sockets clean up env
  - https://github.com/php/php-src/blob/b0b416b705f5b535d95dc7d275347f89f3ef87ea/sapi/fpm/fpm/fpm_sockets.c#L71
- **Feat**: pool - look to introducing pool manager process handlig - reduce load on master and better (possibly more secure) separation and reload
  - https://bugs.php.net/bug.php?id=75440 - Fpm reload should be graceful, not killing running processes (possibly master could just re-read config and let proc mangers deal with it
  - https://bugs.php.net/bug.php?id=60961 - Graceful Restart (USR2) isn't very graceful (similar to above bug listing problems with graceful reload)
  - https://github.com/php/php-src/pull/3758 - php-fpm: graceful restart without blocking/losing requests (Mike's old PR with some useful discussion about reload)
  - https://bugs.php.net/bug.php?id=74778 - opcache_clear() puts php-fpm master in a cpu loop
  - https://bugs.php.net/bug.php?id=74709 - PHP-FPM process eating 100% CPU attempting to use kill_all_lockers (separating opache MINIT)
  - https://github.com/php/php-src/issues/8072 - php-fpm trying to kill another user's pool results in an infinite loop and 99% cpu usage (separating opache MINIT)
- **Feat**: pool - look to the alternative way of dynamically loading pool configuration and restarting it
  - https://bugs.php.net/bug.php?id=61595 - implement dynamic loading of pools config via file or SQL
  - https://bugs.php.net/bug.php?id=51973 - a way to restart single pools, enable/disable modules per pool
- **Feat**: pool / proc - control group (;linux namespace) support
  - https://bugs.php.net/bug.php?id=80657 - Linux namespace support
  - https://bugs.php.net/bug.php?id=70605 - Option to attach a pool to a cgroup

## Feedback required


- https://github.com/php/php-src/issues/8646 - Core: Memory leak PHP FPM 8.1 ARM64
- https://bugs.php.net/bug.php?id=73313 - over fpm does not respect in .user.ini engine off directive
- https://bugs.php.net/bug.php?id=76224 - Error and shutdown handlers triggered on object destroy
- https://bugs.php.net/bug.php?id=70945 - php-fpm don't start : ERROR: no data have been read from pipe failed
- https://bugs.php.net/bug.php?id=67589 - With --nodaemonize errors don't get into error_log directive

## Testing


- **Test**: signals - Rename and clean up the signal reloading tests
  - bug74085-concurrent-reload.phpt
  - bug76601-reload-child-signals.phpt
- **Test**: proc - try to make sigquit test work
  - https://github.com/php/php-src/issues/8443 - sapi/fpm/tests/bug77023-pm-dynamic-blocking-sigquit.phpt test is failing frequently
- **Test**: proc - make proc idle test stable
  - https://github.com/php/php-src/pull/7561#issuecomment-980802605
- **Test**: proc - skip ondemand tests on unsupported platforms
  - https://bugs.php.net/bug.php?id=81110 - Tests use unsupported ondemand process manager
- **Test**: logs - Make log-bwd-multiple-msgs-stdout-stderr work again
- **Test**: socket - Check why bug68381 test fails with ASAN (currently skip with ASAN)
- **Test**: fcgi - Rename and clean up HTTP Status header truncation
- **Test**: binary - better integrate exit code test - bug78323.phpt


## Docs

- extend access log documentation - better document %e options for example
  - https://bugs.php.net/bug.php?id=62828 - Need documentation for access.format tokens
- proc - improve docs for pm directives - create a new section on process management
  - https://bugs.php.net/bug.php?id=81242 - Unclear documentation for process management directives


## Changes

### 2023-02

- **Bug**: main / fcgi - investigate if file uploads can cause disk space exhaustion
  - https://bugs.php.net/bug.php?id=75889 - Overloading disk with temporary files
  - https://bugs.php.net/bug.php?id=70620 php-fpm can interrupt sapi_deactivate cleanup (Security bug - exposed by the #75889)
- **Bug**: event - Re-classify unknown child alert
  - https://github.com/php/php-src/issues/10315 - FPM unknown child alert not valid
  - https://github.com/php/php-src/issues/10258 - Unknown child (SOME_ID)
  - https://github.com/php/php-src/issues/9792 - ALERT: oops, unknown child (103) exited with code 0
  - https://github.com/php/php-src/issues/9438 - ALERT: oops, unknown child (?) exited with code 32
- **Bug**: scoreboard - slow request causing counter reset
  - https://bugs.php.net/bug.php?id=70645 - Resetting counters on status page
### 2023-01

- **Bug**: proc - fix the comment in www.conf to better clarify user, group, listen.user and listen.group (possibly also check docs correctness)
  - https://bugs.php.net/bug.php?id=67244 - Wrong owner:group for listening unix socket
- **Bug**: main - ASAN fixes

### 2022-12

- **Bug**: proc - verify user existence when runnin `php-fpm -t`
  - https://bugs.php.net/bug.php?id=68591 - Configuration test does not perform uid lookups
  - https://bugs.php.net/bug.php?id=75057 - php-fpm -t doesn't verify does user exist (duplicate)
- **Bug**: main - fastcgi.error_header does not change the header for subsequent requests
  - https://github.com/php/php-src/issues/9981 - FPM does not reset fastcgi.error_header
- **Bug**: scoreboard - inconsistent idle process count
  - https://bugs.php.net/bug.php?id=74542 - Inconsistent php-fpm status page
- **Bug**: error log - missing new line
  - https://bugs.php.net/bug.php?id=77106 - Missing newlines between PHP messages

### 2022-11

- **Bug**: The fpm_stdio_child_said can happen after the child is deleted
  - https://github.com/php/php-src/issues/8517 - Random crash of php8 - segfault...
  - https://github.com/php/php-src/pull/9444 - Fix GH-8517: FPM child pointer can be possibly uninitialized
- **Bug**: proc - initgroups fails if user numeric (uid)
  - https://bugs.php.net/bug.php?id=80669 - Can't initgroups() when specifying numeric user
- **Bug**: main - fastcgi.error_header sets header after request is sent
  - https://bugs.php.net/bug.php?id=68207 - Setting fastcgi.error_header can result in an E_WARNING being triggered
  - https://github.com/php/php-src/pull/9980 - Fix bug #68207: Setting fastcgi.error_header can result in a WARNING

### 2022-10

- **Bug**: SaltStack no longer works for PHP-8.0
  - https://github.com/php/php-src/issues/9754 - SaltStack (using Python subprocess) hangs when running php-fpm 8.1.11

### 2022-09

- **Test**: tester - Reworked logging for better debugging
  - https://github.com/php/php-src/pull/9588 - Rework FPM tests logging for better debugging
    - allow out of order log entries with some being optional and debug only collected (some sort of event based collector)
    - print all logs when error including some tracing info if possible (change log levels to debug)
### 2022-08

- **Bug**: main - incorrectly marking headers as sent when previous connection aborted
  - https://bugs.php.net/bug.php?id=77780 - "Headers already sent..." when previous connection was aborted

### 2022-06

- **Bug**: syslog - fix syslog ident issues
  - https://github.com/php/php-src/pull/7334 - fix for 81324
  - https://bugs.php.net/bug.php?id=81324 - php-fpm will not set syslog ident for children
  - https://bugs.php.net/bug.php?id=74469 - syslog.ident setting ignored when logging errors (error_log=syslog)
  - https://bugs.php.net/bug.php?id=67764 - fpm: syslog.ident don't work (truncated ?)

### 2022-05

- **Bug**: fcgi - Fix issue with content-length 0 causing nginx 502
  - https://bugs.php.net/bug.php?id=72185 - php-fpm writes stdout with contentlength =0 causing nginx 502
  - https://github.com/php/php-src/pull/3198 - fix - needs a test

### 2022-04

- **Feat**: socket - change default for listen.backlog on Linux to -1
  - https://github.com/php/php-src/pull/8410 - fpm: listen backlog should default to -1 also on Linux (reviewed and merged)
- **Bug**: proc - look to the issue with dynamic mode leaving too many idle children
  - https://bugs.php.net/bug.php?id=77023 - PHP-FPM cannot shutdown processes
- **Bug**: scoreboard - active processes above max_children (possibly related to lock issue above)
  - https://bugs.php.net/bug.php?id=76003 - FPM /status reports wrong number of active processes
- **Feat**: socket - emit error for invalid port setting (reviewed, tested and merged)
  - https://github.com/php/php-src/pull/8225 - fpm warns about port settings
- **Feat**: selinux dumpable (reviewed and merged)
  - https://github.com/php/php-src/pull/7648 - fpm dumpable process setting extra check for SElinux based systems

### 2022-03

- **Doc**: configuration - missing settings - review also as there are more missing
  - https://bugs.php.net/bug.php?id=67094 - Missing FPM settings in documentation (already done so just closed)
  - https://bugs.php.net/bug.php?id=63888 - Missing several PHP FPM ini entries in online documentation (already done so just closed)
  - missing values:
    - pm.max_spawn_rate - PHP 8.1
    - pm.status_listen - PHP 8.0
    - request_slowlog_trace_depth - PHP 7.2
    - request_terminate_timeout_track_finished - PHP
    - apparmor_hat
- **Doc**: socket - documentation listen.allowed_clients empty value (`any` is not correct name)
  - https://bugs.php.net/bug.php?id=80580 - listen.allowed_clients does not operate as expected with blank or "any" value
- **Doc**: socket - clarify docs for listen backlog values
  - https://bugs.php.net/bug.php?id=78611 - listen.backlog is not -1, and -1 does not mean 'unlimited'.

### 2022-02

- **Bug**: scoreboard - fixed locking of getting proc info
  - https://bugs.php.net/bug.php?id=76109 - Unsafe access to fpm scoreboard
  - https://bugs.php.net/bug.php?id=81275 - status page, contain bugus value in request duration sometimes
  - https://github.com/php/php-src/pull/3188 - Implement fpm_scoreboard_copy and use it for fpm_handle_status_request
  - https://github.com/php/php-src/pull/8049 - Implement fpm_scoreboard_copy (extended rebase of 3188)
  - https://github.com/php/php-src/issues/7931 - PHP-FPM worker process stuck after slow log tracing
- **Doc**: document status page
  - https://bugs.php.net/bug.php?id=72105 - Missing Documentation: PHP-FPM status page
  - https://github.com/php/doc-en/pull/738 - Documentation for the FPM status page and fpm_get_status()
  - https://github.com/php/doc-en/pull/1420 - FPM Status Page
