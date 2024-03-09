[TOC]
net/http https://godoc.org/net/http
&nbsp;&nbsp; abc
&nbsp;&nbsp;&nbsp;&nbsp; abc
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; abc
#  Request
``` go
type Request
    func NewRequest(method, url string, body io.Reader) (*Request, error)
    func ReadRequest(b *bufio.Reader) (*Request, error)
    func (r *Request) AddCookie(c *Cookie)
    func (r *Request) BasicAuth() (username, password string, ok bool)
    func (r *Request) Context() context.Context
    func (r *Request) Cookie(name string) (*Cookie, error)
    func (r *Request) Cookies() []*Cookie
    func (r *Request) FormFile(key string) (multipart.File, *multipart.FileHeader, error)
    func (r *Request) FormValue(key string) string
    func (r *Request) MultipartReader() (*multipart.Reader, error)
    func (r *Request) ParseForm() error
    func (r *Request) ParseMultipartForm(maxMemory int64) error
    func (r *Request) PostFormValue(key string) string
    func (r *Request) ProtoAtLeast(major, minor int) bool
    func (r *Request) Referer() string
    func (r *Request) SetBasicAuth(username, password string)
    func (r *Request) UserAgent() string
    func (r *Request) WithContext(ctx context.Context) *Request
    func (r *Request) Write(w io.Writer) error
    func (r *Request) WriteProxy(w io.Writer) error
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
#  Response
``` go
type Response
    func Get(url string) (resp *Response, err error)
    func Head(url string) (resp *Response, err error)
    func Post(url, contentType string, body io.Reader) (resp *Response, err error)
    func PostForm(url string, data url.Values) (resp *Response, err error)
    func ReadResponse(r *bufio.Reader, req *Request) (*Response, error)
    func (r *Response) Cookies() []*Cookie
    func (r *Response) Location() (*url.URL, error)
    func (r *Response) ProtoAtLeast(major, minor int) bool
    func (r *Response) Write(w io.Writer) error
```
``` go
type Response struct {
	Status     string // e.g. "200 OK"
	StatusCode int    // e.g. 200
	Proto      string // e.g. "HTTP/1.0"
	ProtoMajor int    // e.g. 1
	ProtoMinor int    // e.g. 0
	Header Header
	Body io.ReadCloser
	ContentLength int64
	TransferEncoding []string
	Close bool
	Uncompressed bool
	Trailer Header
	Request *Request
	TLS *tls.ConnectionState
}
```
#   Header
``` go
type Header
    func (h Header) Add(key, value string)
    func (h Header) Del(key string)
    func (h Header) Get(key string) string
    func (h Header) Set(key, value string)
    func (h Header) Write(w io.Writer) error
    func (h Header) WriteSubset(w io.Writer, exclude map[string]bool) error
```
``` go
type Header map[string][]string
```
#   Cookie
``` go
type Cookie struct {
	Name  string
	Value string
	Path       string    // optional
	Domain     string    // optional
	Expires    time.Time // optional
	RawExpires string    // for reading cookies only
	MaxAge   int
	Secure   bool
	HttpOnly bool
	SameSite SameSite
	Raw      string
	Unparsed []string // Raw text of unparsed attribute-value pairs
}
```
``` go
type Cookie
    func (c *Cookie) String() string
```
#   url.URL
``` go
type URL
    func Parse(rawurl string) (*URL, error)
    func ParseRequestURI(rawurl string) (*URL, error)
    func (u *URL) EscapedPath() string
    func (u *URL) Hostname() string
    func (u *URL) IsAbs() bool
    func (u *URL) MarshalBinary() (text []byte, err error)
    func (u *URL) Parse(ref string) (*URL, error)
    func (u *URL) Port() string
    func (u *URL) Query() Values
    func (u *URL) RequestURI() string
    func (u *URL) ResolveReference(ref *URL) *URL
    func (u *URL) String() string
    func (u *URL) UnmarshalBinary(text []byte) error
```
``` go
type URL struct {
	Scheme     string
	Opaque     string    // encoded opaque data
	User       *Userinfo // username and password information
	Host       string    // host or host:port
	Path       string    // path (relative paths may omit leading slash)
	RawPath    string    // encoded path hint (see EscapedPath method)
	ForceQuery bool      // append a query ('?') even if RawQuery is empty
	RawQuery   string    // encoded query values, without '?'
	Fragment   string    // fragment for references, without '#'
}
```
#   server
``` go
type Server
    func (srv *Server) Close() error
    func (srv *Server) ListenAndServe() error
    func (srv *Server) ListenAndServeTLS(certFile, keyFile string) error
    func (srv *Server) RegisterOnShutdown(f func())
    func (srv *Server) Serve(l net.Listener) error
    func (srv *Server) ServeTLS(l net.Listener, certFile, keyFile string) error
    func (srv *Server) SetKeepAlivesEnabled(v bool)
    func (srv *Server) Shutdown(ctx context.Context) error
```
``` go
type Server struct {
	Addr string
	Handler Handler // handler to invoke, http.DefaultServeMux if nil
	TLSConfig *tls.Config
	ReadTimeout time.Duration
	ReadHeaderTimeout time.Duration
	WriteTimeout time.Duration
	IdleTimeout time.Duration
	MaxHeaderBytes int
	TLSNextProto map[string]func(*Server, *tls.Conn, Handler)
	ConnState func(net.Conn, ConnState)
	ErrorLog *log.Logger
	BaseContext func(net.Listener) context.Context
	ConnContext func(ctx context.Context, c net.Conn) context.Context
	disableKeepAlives int32     // accessed atomically.
	inShutdown        int32     // accessed atomically (non-zero means we're in Shutdown)
	nextProtoOnce     sync.Once // guards setupHTTP2_* init
	nextProtoErr      error     // result of http2.ConfigureServer if used
	mu         sync.Mutex
	listeners  map[*net.Listener]struct{}
	activeConn map[*conn]struct{}
	doneChan   chan struct{}
	onShutdown []func()
}
```
``` go
type ServeMux struct {
	mu    sync.RWMutex
	m     map[string]muxEntry
	es    []muxEntry // slice of entries sorted from longest to shortest.
	hosts bool       // whether any patterns contain hostnames
}
```
``` go
type Handler interface {
	ServeHTTP(ResponseWriter, *Request)
}

type HandlerFunc func(ResponseWriter, *Request)

func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) {
	f(w, r)
}
```
路由注册
&nbsp;&nbsp; func HandleFunc(pattern string, handler func(ResponseWriter, *Request))
&nbsp;&nbsp;&nbsp;&nbsp; ServeMux.HandleFunc
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ServeMux.Handle
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; add to router map

&nbsp;&nbsp; func Handle(pattern string, handler Handler)
&nbsp;&nbsp;&nbsp;&nbsp; ServeMux.Handle

处理请求
&nbsp;&nbsp; Server.ListenAndServe
&nbsp;&nbsp;&nbsp;&nbsp; net.Listen
&nbsp;&nbsp;&nbsp;&nbsp; Server.Serve
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; net.Listener.Accept
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; baseCtx := context.Background()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ctx := context.WithValue(baseCtx, ServerContextKey, srv)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; go conn.serve(ctx)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ctx = context.WithValue(ctx, LocalAddrContextKey, c.rwc.LocalAddr())
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ctx, cancelCtx := context.WithCancel(ctx)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; conn.cancelCtx = cancelCtx
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; defer cancelCtx()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; conn.readRequest
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ctx, cancelCtx := context.WithCancel(ctx)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Request.ctx = ctx
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; response{Request, cancelCtx}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; serverHandler.ServeHTTP
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ServeMux.ServeHTTP
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ServeMux.Handler
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Handler.ServeHTTP
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; response.cancelCtx()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; response.finishRequest()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; response.reqBody.Close()
#   client
``` go
type Client
    func (c *Client) CloseIdleConnections()
    func (c *Client) Do(req *Request) (*Response, error)
    func (c *Client) Get(url string) (resp *Response, err error)
    func (c *Client) Head(url string) (resp *Response, err error)
    func (c *Client) Post(url, contentType string, body io.Reader) (resp *Response, err error)
    func (c *Client) PostForm(url string, data url.Values) (resp *Response, err error)
```
``` go
type Client struct {
	Transport RoundTripper
	CheckRedirect func(req *Request, via []*Request) error
	Jar CookieJar
	Timeout time.Duration
}
```
``` go
type RoundTripper interface {
	RoundTrip(*Request) (*Response, error)
}

type Transport struct {
	idleMu       sync.Mutex
	closeIdle    bool                                // user has requested to close all idle conns
	idleConn     map[connectMethodKey][]*persistConn // most recently used at end
	idleConnWait map[connectMethodKey]wantConnQueue  // waiting getConns
	idleLRU      connLRU
	reqMu       sync.Mutex
	reqCanceler map[*Request]func(error)
	altMu    sync.Mutex   // guards changing altProto only
	altProto atomic.Value // of nil or map[string]RoundTripper, key is URI scheme
	connsPerHostMu   sync.Mutex
	connsPerHost     map[connectMethodKey]int
	connsPerHostWait map[connectMethodKey]wantConnQueue // waiting getConns
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
	WriteBufferSize int
	ReadBufferSize int
	nextProtoOnce      sync.Once
	h2transport        h2Transport // non-nil if http2 wired up
	tlsNextProtoWasNil bool        // whether TLSNextProto was nil when the Once fired
	ForceAttemptHTTP2 bool
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
&nbsp;&nbsp; NewRequest/NewRequestWithContext
&nbsp;&nbsp; Client.Do
&nbsp;&nbsp;&nbsp;&nbsp; Client.do
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Client.send
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; send
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; RoundTripper.RoundTrip
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Transport.roundTrip
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ctx := Request.Context()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Transport.getConn
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; ctx := req.Context()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; persistConn.roundTrip
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Request.closeBody
#   feature
##  body
### NopCloser
``` go
	rc, ok := body.(io.ReadCloser)
	if !ok && body != nil {
		rc = ioutil.NopCloser(body)
	}
```
### Body Dump
``` go
    // 把request的内容读取出来
    var bodyBytes []byte
    if c.Request.Body != nil {
        bodyBytes, _ = ioutil.ReadAll(c.Request.Body)
    }
    // 把刚刚读出来的再写进去
    c.Request.Body = ioutil.NopCloser(bytes.NewBuffer(bodyBytes))
```
### Request
#### server
The Server will close the request body. The ServeHTTP Handler does not need to.
t.Body = &body{src: io.LimitReader(r, realLength), closing: t.Close}
finishRequest -> w.reqBody.Close()
#### client
HTTP Client's Transport is responsible for calling the Close method.
Client.do -> req.closeBody()
### Response
It is the caller's responsibility to close Body.
#### server
没有用到Response，ResponseWriter->response
#### client
t.Body = &body{src: io.LimitReader(r, realLength), closing: t.Close}
defer resp.Body.Close()
## context & cancel
### client
As background, there are three ways to cancel a request:
First was Transport.CancelRequest. (deprecated)
Second was Request.Cancel (deprecated).
Third was Request.Context.
*	Transport.CancelRequest(deprecated)
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
```
*	Request.Cancel
``` go
// Cancel is an optional channel whose closure indicates that the client
// request should be regarded as canceled. Not all implementations of
// RoundTripper may support Cancel.
//
// For server requests, this field is not applicable.
//
// Deprecated: Set the Request's context with NewRequestWithContext
// instead. If a Request's Cancel field and context are both
// set, it is undefined whether Cancel is respected.
Cancel <-chan struct{}
```
*	Request.Context
func NewRequestWithContext(ctx context.Context, method, url string, body io.Reader) (*Request, error)
``` go
// ctx is either the client or server context. It should only
// be modified via copying the whole Request using WithContext.
// It is unexported to prevent people from using Context wrong
// and mutating the contexts held by callers of the same request.
ctx context.Context
```
### server
read/write timeout + 业务Context
#	usage
##	server
###	timeout
type Server struct {
	ReadTimeout time.Duration
	ReadHeaderTimeout time.Duration
	WriteTimeout time.Duration
	IdleTimeout time.Duration
###	graceful shutdown
``` go
func (srv *Server) Shutdown(ctx context.Context) error {
	atomic.StoreInt32(&srv.inShutdown, 1)

	srv.mu.Lock()
	lnerr := srv.closeListenersLocked()
	srv.closeDoneChanLocked()
	for _, f := range srv.onShutdown {
		go f()
	}
	srv.mu.Unlock()

	ticker := time.NewTicker(shutdownPollInterval)
	defer ticker.Stop()
	for {
		if srv.closeIdleConns() {
			return lnerr
		}
		select {
		case <-ctx.Done():
			return ctx.Err()
		case <-ticker.C:
		}
	}
}
```
##	client
###	超时
type Client struct {
	Timeout time.Duration
###	连接池
type Transport struct {
	MaxIdleConns int
	MaxIdleConnsPerHost int
	MaxConnsPerHost int
	IdleConnTimeout time.Duration
##	ReverseProxy
## 跨域