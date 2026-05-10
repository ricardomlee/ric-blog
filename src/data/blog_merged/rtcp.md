---
description: Popular RTCP implementations.
pubDatetime: 2023-07-23
tags:
- Network
title: RTCP协议的不同实现对比（RIST vs WebRTC）
---

## RFC 3550

[RFC 3550: RTP: A Transport Protocol for Real-Time Applications (rfc-editor.org)](https://www.rfc-editor.org/rfc/rfc3550)

```
6.7 APP: Application-Defined RTCP Packet

    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |V=2|P| subtype |   PT=APP=204  |             length            |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                           SSRC/CSRC                           |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                          name (ASCII)                         |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                   application-dependent data                ...
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

## Standards

1. RIST
    1. [**VSF_TR-06-1_2018_10_17.pdf**](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&ved=2ahUKEwilwpq9gcqAAxVmPkQIHbFxAoIQFnoECA8QAQ&url=https%3A%2F%2Fvsf.tv%2Fdownload%2Ftechnical_recommendations%2FVSF_TR-06-1_2018_10_17.pdf&usg=AOvVaw33l3pjZUCsf596K-2rquPw&opi=89978449)
2. WebRTC
    1. [RFC 8825: Overview: Real-Time Protocols for Browser-Based Applications (rfc-editor.org)](https://www.rfc-editor.org/rfc/rfc8825)
    

## Source code

1. librist
    1. [rist / librist · GitLab (videolan.org)](https://code.videolan.org/rist/librist)
        1. receiver_mark_missing
        2. 没有处理乱序
        3. 根据包到来的时间计算nack发送时间
        4. 添加到missing队列
        
        ```jsx
        static inline void receiver_mark_missing(struct rist_flow *f, struct rist_peer *peer, uint32_t current_seq, uint64_t rtt) {
        	uint32_t counter = 1;
        	uint64_t packet_time_last = 0;
        	if (RIST_UNLIKELY(!f->receiver_queue[f->last_seq_found]))
        		if (RIST_LIKELY(!f->rtc_timing_mode))
        			packet_time_last = timestampNTP_u64();
        		else
        			packet_time_last = timestampNTP_RTC_u64();
        	else
        		packet_time_last = f->receiver_queue[f->last_seq_found]->packet_time;
        	uint64_t packet_time_now = f->receiver_queue[current_seq]->packet_time;
        	uint32_t missing_count = (current_seq - f->last_seq_found) & UINT16_MAX;
        	//arbitrary large number to prevent incorrectly marking packets as missing when wrap-around occurs & we did not correctly detect as out of order
        	if (missing_count > 32768)
        		return;
        	uint64_t interpacket_time = (packet_time_now - packet_time_last) / (missing_count +1);
        	uint32_t missing_seq = (f->last_seq_found + counter);
        
        	if (f->short_seq)
        		missing_seq = (uint16_t)missing_seq;
        
        	uint64_t nack_time = packet_time_last;
        	while (missing_seq != current_seq)
        	{
        		nack_time += interpacket_time;
        		if (RIST_UNLIKELY(peer->buffer_bloat_active || f->missing_counter > peer->missing_counter_max))
        		{
        			if (f->missing_counter > peer->missing_counter_max)
        				rist_log_priv(get_cctx(peer), RIST_LOG_DEBUG,
        					"Retry buffer is already too large (%d) for the configured "
        					"bandwidth ... ignoring missing packet(s).\n",
        					f->missing_counter);
        			if (peer->buffer_bloat_active)
        				rist_log_priv(get_cctx(peer), RIST_LOG_ERROR,
        					"Link has collapsed. Not queuing new retries until it recovers.\n");
        			break;
        		}
        		rist_receiver_missing(f, peer, nack_time, missing_seq, rtt);
        		if (RIST_UNLIKELY(counter == f->receiver_queue_max))
        			break;
        		counter++;
        		missing_seq = (f->last_seq_found + counter);
        		if (f->short_seq)
        			missing_seq = (uint16_t)missing_seq;
        	}
        }
        
        ...
        
        void rist_receiver_missing(struct rist_flow *f, struct rist_peer *peer,uint64_t nack_time, uint32_t seq, uint64_t rtt)
        {
        	struct rist_missing_buffer *m = calloc(1, sizeof(*m));
        	uint64_t now = timestampNTP_u64();
        	if (nack_time > now)
        		nack_time = now;
        	if (nack_time < (now - f->recovery_buffer_ticks))
        		nack_time = now;
        	m->seq = seq;
        	m->insertion_time = nack_time;
        
        	m->next_nack = now + rtt;
        	m->peer = peer;
        
        	if (get_cctx(peer)->debug)
        		rist_log_priv(get_cctx(peer), RIST_LOG_DEBUG,
        			"Datagram %" PRIu32 " is missing, inserting into the missing queue "
        			"with deadline in %" PRIu64 "ms (queue=%d), last_seq_found %"PRIu32"\n",
        		seq, m->next_nack > now? (m->next_nack - now)/ RIST_CLOCK: 0, f->missing_counter, f->last_seq_found);
        
        	m->next = NULL;
        	// Insert it at the end of the queue
        	if (!f->missing) {
        		f->missing = m;
        		f->missing_tail = m;
        	} else {
        		f->missing_tail->next = m;
        		f->missing_tail = m;
        	}
        }
        ```
        
2. webrtc-rs
    1. [webrtc-rs/webrtc: A pure Rust implementation of WebRTC (github.com)](https://github.com/webrtc-rs/webrtc)
        1. 丢包检测：[webrtc/interceptor/src/nack/generator/generator_stream.rs at master · webrtc-rs/webrtc (github.com)](https://github.com/webrtc-rs/webrtc/blob/master/interceptor/src/nack/generator/generator_stream.rs)