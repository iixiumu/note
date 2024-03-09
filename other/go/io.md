strings
	func NewReader(s string) *Reader
bytes
	func NewReader(b []byte) *Reader

    func NewBuffer(buf []byte) *Buffer
    func NewBufferString(s string) *Buffer
    func (b *Buffer) Bytes() []byte
    func (b *Buffer) String() string
bufio
	func NewReadWriter(r *Reader, w *Writer) *ReadWriter
	func NewReader(rd io.Reader) *Reader
	func NewWriter(w io.Writer) *Writer
io.ioutil
	func NopCloser(r io.Reader) io.ReadCloser

io
type Reader interface {
    Read(p []byte) (n int, err error)
}
type Writer interface {
    Write(p []byte) (n int, err error)
}
type ReadWriter interface {
    Reader
    Writer
}
type Closer interface {
    Close() error
}
type ReadCloser interface {
    Reader
    Closer
}
type WriteCloser interface {
    Writer
    Closer
}
type ReadWriteCloser interface {
    Reader
    Writer
    Closer
}

func LimitReader(r Reader, n int64) Reader