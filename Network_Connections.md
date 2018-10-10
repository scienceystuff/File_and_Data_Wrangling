### Network Connections
This markdown file described several ways to check network connections or create network connections (such as a remote SSH tunnel).  Be sure to check the comments for each command for notes on compatibillity across various *nix platforms.


#### Network Connections Commands

1) Create an SSH connection from YOURCOMPUTER to REMOTECOMPUTER using port number PORTNUM

        ssh -l username -p PORTNUM REMOTECOMPUTER
        # this is equivalent to:
        ssh -p PORTNUM username@REMOTECOMPUTER
 
2) Setup passwordless SSH from YOURCOMPUTER to a REMOTECOMPUTER

        cat ~/.ssh/id_rsa.pub | ssh username@REMOTECOMPUTER 'cat >> ~/.ssh/authorized_keys'
        # Comments: SSH setup (i.e. the 'ssh-keygen' command) should be run on the remote computer before running this command
        
3) Create a remote tunnel from YOURCOMPUTER to REMOTECOMPUTER at port RPORT on the remote machine, which will allow you to access whatever service is running on RPORT using the port LOCALP on YOURCOMPUTER.

        export LOCALP=99999 ; export RPORT=22 ; export REMOTECOMPUTER=remotecomputer.somedomain.com ; export YOURCOMPUTER=localhost ; export LOCALP=22 ; ssh -fNT -l username -L $LOCALP:$REMOTECOMPUTER:$LOCALP $YOURCOMPUTER
        # Comments:
        # # on MacOSX use 'localhost', on Linux use '127.0.0.1'
        # # you must have a valid login on the remote computer, and using passwordless SSH is *strongly* encouraged
        # # the port on your machine that you'll use to access the service on the remote computer is LOCALP
        # # example: after running the above command, type ssh -p $LOCALPORT username@localhost to access SSH on the remote machine
        # # the port on the remote machine RPORT must be available and not already in use
        # # the -f option send the shell to the background before remote command execution
        # # the -N option instructs SSH not to execute any remote commands
        # # the -T option disables pseudo-tty allocation

4) See who is logged in on a remote computer REMOTECOMPUTER; assumes that you have the login *username*

        ssh username@REMOTECOMPUTER 'who'
        # will print the remote command to STDOUT
        
5) See a list of all TCP/UDP connections, and optionally filter with 'ACCEPTED' or 'ESTABLISHED'

        # Linux: search for all connections on port PORT, note that running as sudo is recommended
        netstat -nltp | grep PORT
        # MacOSX: search for all tcp traffic on PORT, note that running as sudo is recommended
        lsof -nP -iTCP:PORT
        # or
        lsof -nP -i:PORT | grep LISTEN
        
6) Show a possible path taken by REMOTEHOST to connect to YOURCOMPUTER

        traceroute REMOTEHOST
        # Comments:
        # # using nmap is strongly recommended instead of this command
        # # in Linux you usually have to be sudo/root to run this command
        
7) Lookup the fully qualified domain name FQDN or IP address IPADDR for REMOTECOMPUTER

        dig FQDN
        # or do a reverse lookup
        dig -x IPADDR
        
