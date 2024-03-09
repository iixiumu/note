#	Request
``` go
type Request struct {
	Method string
	URL *url.URL
	Proto      string // "HTTP/1.0"
	ProtoMajor int    // 1
	ProtoMinor int    // 0
	Header Header
*	Body io.ReadCloser
	GetBody func() (io.ReadCloser, error)
	ContentLength int64
	TransferEncoding []string
	Close bool
	Host string
	Form url.Values
	PostForm url.Values
	MultipartForm *multipart.Form
	Trailer Header
	RemoteAddr string
	RequestURI string
	TLS *tls.ConnectionState
*	Cancel <-chan struct{}
	Response *Response
*	ctx context.Context
}
```
#	server
``` go
type Server struct {
	Addr    string  // TCP address to listen on, ":http" if empty
	Handler Handler // handler to invoke, http.DefaultServeMux if nil
*	ReadTimeout time.Duration
*	ReadHeaderTimeout time.Duration
*	WriteTimeout time.Duration
*	IdleTimeout time.Duration
	MaxHeaderBytes int
	TLSNextProto map[string]func(*Server, *tls.Conn, Handler)
	ConnState func(net.Conn, ConnState)
	ErrorLog *log.Logger
	disableKeepAlives int32     // accessed atomically.
	inShutdown        int32     // accessed atomically (non-zero means we're in Shutdown)
	nextProtoOnce     sync.Once // guards setupHTTP2_* init
	nextProtoErr      error     // result of http2.ConfigureServer if used
	mu         sync.Mutex
*	listeners  map[*net.Listener]struct{}
*	activeConn map[*conn]struct{}
*	doneChan   chan struct{}
	onShutdown []func()
}

```
*	context
conn.serve生成cancelCtx，传入conn.readRequest，生成新的cancelCtx，放入response.req.ctx和response.cancelCtx
Request
``` go
func (c *conn) serve(ctx context.Context) {
	defer func() {
		if !c.hijacked() {
			c.close()
			c.setState(c.rwc, StateClosed)
		}
	}()
	
	ctx, cancelCtx := context.WithCancel(ctx)
	c.cancelCtx = cancelCtx
	defer cancelCtx()
	
	for {
		w, err := c.readRequest(ctx)
		serverHandler{c.server}.ServeHTTP(w, w.req)
		w.cancelCtx()
		if c.hijacked() {
			return
		}
	}
```
*	timeout
	*   ReadHeaderTimeout
	*   ReadTimeout
	*   WriteTimeout
	*   IdleTimeout
*	transport
*	body
``` go
func ListenAndServe(addr string, handler Handler) error
	func (srv *Server) ListenAndServe() error
		ln, err := net.Listen("tcp", addr)
		func (srv *Server) Serve(l net.Listener) error
			for {
				rw, e := l.Accept()

```


``` go
func (c *conn) serve(ctx context.Context) {
    ctx, cancelCtx := context.WithCancel(ctx)
	c.cancelCtx = cancelCtx
    defer cancelCtx()

    for {
        w, err := c.readRequest(ctx)
            if d := c.server.readHeaderTimeout(); d != 0 {
                hdrDeadline = t0.Add(d)
            }
            if d := c.server.ReadTimeout; d != 0 {
                wholeReqDeadline = t0.Add(d)
            }
            c.rwc.SetReadDeadline(hdrDeadline)
            if d := c.server.WriteTimeout; d != 0 {
                defer func() {
                    c.rwc.SetWriteDeadline(time.Now().Add(d))
                }()
            }
                func (c *conn) SetReadDeadline(t time.Time)
                    func (fd *netFD) SetReadDeadline(t time.Time)
                        func (fd *FD) SetReadDeadline(t time.Time)
                            setDeadlineImpl(fd, t, 'r')
                                runtime_pollSetDeadline(fd.pd.runtimeCtx, d, mode)
                func (c *conn) SetWriteDeadline(t time.Time)
                    func (fd *netFD) SetWriteDeadline(t time.Time)
                        func (fd *FD) SetWriteDeadline(t time.Time)
                            setDeadlineImpl(fd, t, 'w')
                                runtime_pollSetDeadline(fd.pd.runtimeCtx, d, mode)

            req, err := readRequest(c.bufr, keepHostHeader)
                err = readTransfer(req, b)
            ctx, cancelCtx := context.WithCancel(ctx)
            req.ctx = ctx
            if !hdrDeadline.Equal(wholeReqDeadline) {
                c.rwc.SetReadDeadline(wholeReqDeadline)
            }
            w = &response{
                cancelCtx:     cancelCtx,
                req:           req,
            }

        serverHandler{c.server}.ServeHTTP(w, w.req)
        w.cancelCtx()

        if d := c.server.idleTimeout(); d != 0 {
			c.rwc.SetReadDeadline(time.Now().Add(d))
			if _, err := c.bufr.Peek(4); err != nil {
				return
			}
		}
		c.rwc.SetReadDeadline(time.Time{})

```
#   client
``` go
type Client struct {
	Transport RoundTripper
	CheckRedirect func(req *Request, via []*Request) error
	Jar CookieJar
	Timeout time.Duration
}
```
``` go
type Request struct {
	Method string
	URL *url.URL
	Proto      string // "HTTP/1.0"
	ProtoMajor int    // 1
	ProtoMinor int    // 0
	Header Header
	Body io.ReadCloser
	GetBody func() (io.ReadCloser, error)
	ContentLength int64
	TransferEncoding []string
	Close bool
	Host string
	Form url.Values
	PostForm url.Values
	MultipartForm *multipart.Form
	Trailer Header
	RemoteAddr string
	RequestURI string
	TLS *tls.ConnectionState
	Cancel <-chan struct{}
	Response *Response
	ctx context.Context
}
```
``` go
type RoundTripper interface {
	RoundTrip(*Request) (*Response, error)
}
type Transport struct {
	idleMu     sync.Mutex
	wantIdle   bool                                // user has requested to close all idle conns
	idleConn   map[connectMethodKey][]*persistConn // most recently used at end
	idleConnCh map[connectMethodKey]chan *persistConn
	idleLRU    connLRU
	reqMu       sync.Mutex
	reqCanceler map[*Request]func(error)
	altMu    sync.Mutex   // guards changing altProto only
	altProto atomic.Value // of nil or map[string]RoundTripper, key is URI scheme
	connCountMu          sync.Mutex
	connPerHostCount     map[connectMethodKey]int
	connPerHostAvailable map[connectMethodKey]chan struct{}
	Proxy func(*Request) (*url.URL, error)
	DialContext func(ctx context.Context, network, addr string) (net.Conn, error)
	Dial func(network, addr string) (net.Conn, error)
	DialTLS func(network, addr string) (net.Conn, error)
	TLSClientConfig *tls.Config
	TLSHandshakeTimeout time.Duration
	DisableKeepAlives bool
	DisableCompression bool
	MaxIdleConns int
	MaxIdleConnsPerHost int
	MaxConnsPerHost int
	IdleConnTimeout time.Duration
	ResponseHeaderTimeout time.Duration
	ExpectContinueTimeout time.Duration
	TLSNextProto map[string]func(authority string, c *tls.Conn) RoundTripper
	ProxyConnectHeader Header
	MaxResponseHeaderBytes int64
	nextProtoOnce sync.Once
	h2transport   h2Transport // non-nil if http2 wired up
}
```
``` go
type persistConn struct {
	alt RoundTripper
	t         *Transport
	cacheKey  connectMethodKey
	conn      net.Conn
	tlsState  *tls.ConnectionState
	br        *bufio.Reader       // from conn
	bw        *bufio.Writer       // to conn
	nwrite    int64               // bytes written
	reqch     chan requestAndChan // written by roundTrip; read by readLoop
	writech   chan writeRequest   // written by roundTrip; read by writeLoop
	closech   chan struct{}       // closed when conn closed
	isProxy   bool
	sawEOF    bool  // whether we've seen EOF from conn; owned by readLoop
	readLimit int64 // bytes allowed to be read; owned by readLoop
	writeErrCh chan error
	writeLoopDone chan struct{} // closed when write loop ends
	idleAt    time.Time   // time it last become idle
	idleTimer *time.Timer // holding an AfterFunc to close it
	mu                   sync.Mutex // guards following fields
	numExpectedResponses int
	closed               error // set non-nil when conn is closed, before closech is closed
	canceledErr          error // set non-nil if conn is canceled
	broken               bool  // an error has happened on this connection; marked broken so it's not reused.
	reused               bool  // whether conn has had successful request/response and is being reused.
	mutateHeaderFunc func(Header)
}
```






``` go
// CancelRequest cancels an in-flight request by closing its connection.
// CancelRequest should only be called after RoundTrip has returned.
//
// Deprecated: Use Request.WithContext to create a request with a
// cancelable context instead. CancelRequest cannot cancel HTTP/2
// requests.
func (t *Transport) CancelRequest(req *Request) {
	t.cancelRequest(req, errRequestCanceled)
}

// WithContext returns a shallow copy of r with its context changed
// to ctx. The provided ctx must be non-nil.
//
// For outgoing client request, the context controls the entire
// lifetime of a request and its response: obtaining a connection,
// sending the request, and reading the response headers and body.
func (r *Request) WithContext(ctx context.Context) *Request {
	if ctx == nil {
		panic("nil context")
	}
	r2 := new(Request)
	*r2 = *r
	r2.ctx = ctx

	// Deep copy the URL because it isn't
	// a map and the URL is mutable by users
	// of WithContext.
	if r.URL != nil {
		r2URL := new(url.URL)
		*r2URL = *r.URL
		r2.URL = r2URL
	}

	return r2
}

// setRequestCancel sets the Cancel field of req, if deadline is
// non-zero. The RoundTripper's type is used to determine whether the legacy
// CancelRequest behavior should be used.
//
// As background, there are three ways to cancel a request:
// First was Transport.CancelRequest. (deprecated)
// Second was Request.Cancel (this mechanism).
// Third was Request.Context.
func setRequestCancel(req *Request, rt RoundTripper, deadline time.Time) (stopTimer func(), didTimeout func() bool) {
	if deadline.IsZero() {
		return nop, alwaysFalse
	}

	initialReqCancel := req.Cancel // the user's original Request.Cancel, if any

	cancel := make(chan struct{})
	req.Cancel = cancel

	doCancel := func() {
		// The newer way (the second way in the func comment):
		close(cancel)

		// The legacy compatibility way, used only
		// for RoundTripper implementations written
		// before Go 1.5 or Go 1.6.
		type canceler interface {
			CancelRequest(*Request)
		}
		switch v := rt.(type) {
		case *Transport, *http2Transport:
			// Do nothing. The net/http package's transports
			// support the new Request.Cancel channel
		case canceler:
			v.CancelRequest(req)
		}
	}

	stopTimerCh := make(chan struct{})
	var once sync.Once
	stopTimer = func() { once.Do(func() { close(stopTimerCh) }) }

	timer := time.NewTimer(time.Until(deadline))
	var timedOut atomicBool

	go func() {
		select {
		case <-initialReqCancel:
			doCancel()
			timer.Stop()
		case <-timer.C:
			timedOut.setTrue()
			doCancel()
		case <-stopTimerCh:
			timer.Stop()
		}
	}()

	return stopTimer, timedOut.isSet
}
```
``` go
func NewRequest(method, url string, body io.Reader) (*Request, error)
func (c *Client) Do(req *Request) (*Response, error)
    func (c *Client) do(req *Request) (retres *Response, reterr error)
        resp, didTimeout, err = c.send(req, deadline)
            resp, didTimeout, err = send(req, c.transport(), deadline)
                resp, err = rt.RoundTrip(req)
                    func (t *Transport) roundTrip(req *Request) (*Response, error)

```
``` go
func (c *Client) transport() RoundTripper {
	if c.Transport != nil {
		return c.Transport
	}
	return DefaultTransport
}
```