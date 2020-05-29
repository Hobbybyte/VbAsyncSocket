## VbAsyncSocket

Simple and thin WinSock API wrappers for VB6 loosly based on the original [`CAsyncSocket`](https://docs.microsoft.com/en-us/cpp/mfc/reference/casyncsocket-class?view=vs-2017) wrapper in [`MFC`](https://docs.microsoft.com/en-us/cpp/mfc/mfc-and-atl?view=vs-2017).

### Description

Base class `cAsyncSocket` wraps OS non-blocking sockets that can be used to implement various network components in VB6 -- clients and servers -- and supports both async and blocking network communications.

Additionally there is a source-compatible `cTlsSocket` class for transparent TLS transport layer encryption with a couple of crypto backend implementations:

1. Pure VB6 backend with ASM crypto thunks implementation for TLS 1.3 and (legacy) TLS 1.2 client-side and server-side (TLS 1.3 only) support with no dependency on external libraries (like openssl)

2. Native client-side and server-side TLS support using OS provided SSPI library for all available protocol versions.

The VB6 with thunks backend optionally can leverage libsodium primitives for performance reasons (e.g. server-side implementations).

### Usage

Start by including `src\cAsyncSocket.cls` in your project to have a convenient wrapper of most WinSock API functions.

Optionally you can add `src\cTlsSocket.cls` and `src\mdTlsThunks.bas` pair of source files to your project for TLS 1.3 secured connections using VB6 with thunks backend or add `src\cTlsSocket.cls` and `src\mdTlsNative.bas` pair of source files for an alternative backend using native OS provided SSPI library.

### Sample SMTP with STARTTLS

Here is a working sample with error checking omitted for brevity for accessing smtp.gmail.com over port 587.

At first the communication goes over unencrypted plain-text socket, then later it is switched to TLS secured one before issuing the final `QUIT` command.

    With New cTlsSocket
        .SyncConnect "smtp.gmail.com", 587, UseTls:=False
        Debug.Print .SyncReceiveText();
        .SyncSendText "HELO 127.0.0.1" & vbCrLf
        Debug.Print .SyncReceiveText();
        .SyncSendText "STARTTLS" & vbCrLf
        Debug.Print .SyncReceiveText();
        .SyncStartTls "smtp.gmail.com"
        Debug.Print "TLS handshake complete: " & .RemoteHostName
        .SyncSendText "QUIT" & vbCrLf
        Debug.Print .SyncReceiveText();
    End With

Which produces debug output in `Immediate Window` similar to this:
    
    220 smtp.gmail.com ESMTP r3sm6722824lfm.52 - gsmtp
    250 smtp.gmail.com at your service
    220 2.0.0 Ready to start TLS
    Using TLS_CHACHA20_POLY1305_SHA256 from smtp.gmail.com   49158.41 
    Valid ECDSA_SECP256R1_SHA256 signature     49158.42 
    TLS handshake complete: smtp.gmail.com
    221 2.0.0 closing connection r3sm6722824lfm.52 - gsmtp


### ToDo

 - [ ] Allow client to assign client certificate for connection
 - [ ] Provide UI for end-user to choose suitable certificates from Personal certificate store
 - [ ] Add wrappers for http and ftp protocols
 - [x] Add WinSock control replacement
 - [ ] Add more samples (incl. `vbcurl.exe` utility)
 - [ ] Refactor subclassing thunk to use msg queue not to re-enter IDE in debug mode
