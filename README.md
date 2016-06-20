[![Build Status](https://travis-ci.org/ernado/sdp.svg?branch=master)](https://travis-ci.org/ernado/sdp)

# SDP
[RFC 4566](https://tools.ietf.org/html/rfc4566)
SDP: Session Description Protocol in go.

In alpha stage.

### Example
```sdp
v=0
o=jdoe 2890844526 2890842807 IN IP4 10.47.16.5
s=SDP Seminar
i=A Seminar on the session description protocol
u=http://www.example.com/seminars/sdp.pdf
e=j.doe@example.com (Jane Doe)
p=12345
c=IN IP4 224.2.17.12/127
b=CT:154798
t=2873397496 2873404696
r=7d 1h 0 25h
k=clear:ab8c4df8b8f4as8v8iuy8re
a=recvonly
m=audio 49170 RTP/AVP 0
m=video 51372 RTP/AVP 99
b=AS:66781
k=prompt
a=rtpmap:99 h263-1998/90000
```
```go
package main

import (
	"net"
	"time"
	"fmt"

	"github.com/ernado/sdp"
)

func main()  {
	var (
		s sdp.Session
		b []byte
	)
	// defining medias
	audio := sdp.Media{
		Description: sdp.MediaDescription{
			Type:     "audio",
			Port:     49170,
			Format:   "0",
			Protocol: "RTP/AVP",
		},
	}
	video := sdp.Media{
		Description: sdp.MediaDescription{
			Type:     "video",
			Port:     51372,
			Format:   "99",
			Protocol: "RTP/AVP",
		},
		Bandwidths: sdp.Bandwidths{
			sdp.BandwidthApplicationSpecific: 66781,
		},
		Encryption: sdp.Encryption{
			Method: "prompt",
		},
	}
	video.AddAttribute("rtpmap", "99", "h263-1998/90000")

	// defining message
	m := &sdp.Message{
		Origin: sdp.Origin{
			Username:       "jdoe",
			SessionID:      2890844526,
			SessionVersion: 2890842807,
			Address:        "10.47.16.5",
		},
		Name:  "SDP Seminar",
		Info:  "A Seminar on the session description protocol",
		URI:   "http://www.example.com/seminars/sdp.pdf",
		Email: "j.doe@example.com (Jane Doe)",
		Phone: "12345",
		Connection: sdp.ConnectionData{
			IP:  net.ParseIP("224.2.17.12"),
			TTL: 127,
		},
		Bandwidths: sdp.Bandwidths{
			sdp.BandwidthConferenceTotal: 154798,
		},
		Timing: []sdp.Timing{
			{
				Start:  sdp.NTPToTime(2873397496),
				End:    sdp.NTPToTime(2873404696),
				Repeat: 7 * time.Hour * 24,
				Active: 3600 * time.Second,
				Offsets: []time.Duration{
					0,
					25 * time.Hour,
				},
			},
		},
		Encryption: sdp.Encryption{
			Method: "clear",
			Key: "ab8c4df8b8f4as8v8iuy8re",
		},
		Medias: []sdp.Media{audio, video},
	}
	m.AddFlag("recvonly")

	// appending message to session
	s = m.Append(s)

	// appending session to byte buffer
	b = s.AppendTo(b)
	fmt.Println(string(b))
}
```

### Supported params
- [x] v (protocol version)
- [x] o (originator and session identifier)
- [x] s (session name)
- [x] i (session information)
- [x] u (URI of description)
- [x] e (email address)
- [x] p (phone number)
- [x] c (connection information)
- [x] b (zero or more bandwidth information lines)
- [x] t (time)
- [x] r (repeat)
- [x] z (time zone adjustments)
- [x] k (encryption key)
- [x] a (zero or more session attribute lines)
- [x] m (media name and transport address)

### TODO:
- [x] Encoding
- [x] Parsing
- [x] High level encoding
- [x] High level decoding
- [x] Examples
- [ ] More examples and docs
- [x] CI
- [ ] Include to high-level CI

### Possible optimizations
There are comments `// ALLOCATIONS: suboptimal.` and `// CPU: suboptimal. `
that indicate suboptimal implementation that can be optimized. There are often
a benchmarks for this pieces.