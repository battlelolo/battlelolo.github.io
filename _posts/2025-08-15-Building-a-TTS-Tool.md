---
layout: default
title: "Building a TTS Tool for My Friend in One Hour"
date: 2025-08-15
categories: [tech]
tags: [tts, microsoft, ai, text to speech, azure]
---

# Building a TTS Tool for My Friend in One Hour

## The Ask

My friend asked if there was a service that could read academic papers aloud - not like NotebookLM which creates podcast-style summaries, but something that would actually read the original text. She wanted to listen to papers like audiobooks when her eyes got tired.

I didn't know of such a service, but since I'm familiar with Microsoft Azure Language Services, I offered to help: "Send me the paper and I'll make mp3 for you."

## The Reality Check

I thought this would be simple:
1. Extract text from PDF using Claude/ChatGPT/Grok  
2. Run it through Azure TTS (Text to Speech)
3. Done!

Wrong. Academic PDFs are messy. Extract text and you get dozens of co-author names, chart numbers, table data, footnotes - everything my friend didn't want to hear.

I tried asking different AIs to extract only title, abstract, and main content:
- Claude Sonnet: Failed
- Grok: Failed with errors  
- Claude Opus: Success

Turns out extracting clean content from academic PDFs is harder than expected.

## From Manual to Automated

I generated the MP3 using Azure TTS and made a tutorial video. But a week later, my friend was still hunting for paid PDF-to-speech services.

That's when I realized: I might not know the perfect PDF-to-text AI, but I can definitely build a basic TTS web app. 

So I coded one with Claude Sonnet, built it in Next.js, deployed on Vercel, and shared it.

This is why I love being able to code!!!

<img src="{{ site.baseurl }}/assets/images/2025/08/20250815.png" alt="tts" class="img-center">

FYI - Sharing code I used for this service:
jsx'use client';

import { useState } from 'react';

interface StatusState {
  message: string;
  type: 'error' | 'success' | 'loading' | '';
}

export default function Home() {
  const [apiKey, setApiKey] = useState<string>('api-key');
  const [region, setRegion] = useState<string>('eastus');
  const [voice, setVoice] = useState<string>('en-GB-RyanNeural');
  const [rate, setRate] = useState<string>('medium');
  const [text, setText] = useState<string>('Hello, this is Azure Speech text-to-speech demo. How are you today?');
  const [status, setStatus] = useState<StatusState>({ message: '', type: '' });
  const [isPlaying, setIsPlaying] = useState<boolean>(false);
  const [currentAudio, setCurrentAudio] = useState<HTMLAudioElement | null>(null);
  const [lastAudioBlob, setLastAudioBlob] = useState<Blob | null>(null);

  const showStatus = (message: string, type: StatusState['type']) => {
    setStatus({ message, type });
  };

  const testConnection = async () => {
    if (!apiKey || !region) {
      showStatus('API Key와 Region을 입력해주세요.', 'error');
      return;
    }
  };

  const speakLongText = async () => {
    const chunks = splitTextIntoChunks(text);
    
    if (chunks.length === 0) {
      showStatus('텍스트 분할 실패', 'error');
      return;
    }
    
    console.log(`텍스트를 ${chunks.length}개 청크로 분할:`, chunks.map((c, i) => `${i + 1}: ${c.substring(0, 50)}...`));
    showStatus(`긴 텍스트를 ${chunks.length}개 부분으로 나누어 처리 중...`, 'loading');

    const audioChunks: Blob[] = [];
    let accessToken = '';

    try {
      // 액세스 토큰 발급
      console.log('토큰 발급 시도...');
      const tokenResponse = await fetch(`https://${region}.api.cognitive.microsoft.com/sts/v1.0/issueToken`, {
        method: 'POST',
        headers: {
          'Ocp-Apim-Subscription-Key': apiKey
        }
      });

      if (!tokenResponse.ok) {
        const errorText = await tokenResponse.text();
        console.error('토큰 발급 실패:', tokenResponse.status, errorText);
        throw new Error(`토큰 발급 실패: ${tokenResponse.status}`);
      }

      accessToken = await tokenResponse.text();
      console.log('토큰 발급 성공');

      // 각 청크를 순서대로 처리
      for (let i = 0; i < chunks.length; i++) {
        console.log(`청크 ${i + 1}/${chunks.length} 처리 중... (길이: ${chunks[i].length}자)`);
        showStatus(`음성 생성 중... (${i + 1}/${chunks.length})`, 'loading');

        try {
          // SSML에서 특수문자 이스케이프
          const escapedText = chunks[i]
            .replace(/&/g, '&amp;')
            .replace(/</g, '&lt;')
            .replace(/>/g, '&gt;')
            .replace(/"/g, '&quot;')
            .replace(/'/g, '&apos;');

          const ssml = `<speak version='1.0' xml:lang='en-US'><voice xml:lang='en-US' name='${voice}'><prosody rate='${rate}'>${escapedText}</prosody></voice></speak>`;

          const ttsResponse = await fetch(`https://${region}.tts.speech.microsoft.com/cognitiveservices/v1`, {
            method: 'POST',
            headers: {
              'Authorization': 'Bearer ' + accessToken,
              'Content-Type': 'application/ssml+xml',
              'X-Microsoft-OutputFormat': 'audio-16khz-128kbitrate-mono-mp3',
              'User-Agent': 'NextJS-TTS-App'
            },
            body: ssml
          });

          if (!ttsResponse.ok) {
            const errorText = await ttsResponse.text();
            console.error(`청크 ${i + 1} TTS 실패:`, ttsResponse.status, errorText);
            
            // 401 오류시 토큰 재발급 시도
            if (ttsResponse.status === 401) {
              console.log('토큰 재발급 시도...');
              const newTokenResponse = await fetch(`https://${region}.api.cognitive.microsoft.com/sts/v1.0/issueToken`, {
                method: 'POST',
                headers: {
                  'Ocp-Apim-Subscription-Key': apiKey
                }
              });
              
              if (newTokenResponse.ok) {
                accessToken = await newTokenResponse.text();
                i--; // 현재 청크 재시도
                continue;
              }
            }
            
            throw new Error(`TTS 요청 실패 (청크 ${i + 1}): ${ttsResponse.status}`);
          }

          const audioBlob = await ttsResponse.blob();
          
          if (audioBlob.size === 0) {
            console.warn(`청크 ${i + 1}에서 빈 오디오 응답`);
            continue;
          }
          
          console.log(`청크 ${i + 1} 완료, 크기:`, audioBlob.size, 'bytes');
          audioChunks.push(audioBlob);

        } catch (chunkError) {
          console.error(`청크 ${i + 1} 처리 중 오류:`, chunkError);
          showStatus(`청크 ${i + 1} 처리 실패, 계속 진행...`, 'loading');
          continue; // 다음 청크로 계속
        }

        // API 제한을 피하기 위한 지연 (증가)
        if (i < chunks.length - 1) {
          await new Promise(resolve => setTimeout(resolve, 1500));
        }
      }

      if (audioChunks.length === 0) {
        throw new Error('처리된 오디오 청크가 없습니다');
      }

      console.log(`${audioChunks.length}개 청크 완료, 결합 중...`);
      
      // 모든 오디오 청크를 하나로 합치기
      const combinedBlob = new Blob(audioChunks, { type: 'audio/mpeg' });
      console.log('결합된 오디오 크기:', combinedBlob.size, 'bytes');
      setLastAudioBlob(combinedBlob);

      // 재생
      const audioUrl = URL.createObjectURL(combinedBlob);
      const audio = new Audio(audioUrl);
      setCurrentAudio(audio);
      setIsPlaying(true);
      
      audio.addEventListener('canplay', () => {
        console.log('오디오 재생 준비 완료');
      });

      audio.play().catch(playError => {
        console.error('오디오 재생 오류:', playError);
        showStatus('오디오 재생 실패', 'error');
        setIsPlaying(false);
        setCurrentAudio(null);
      });

      showStatus(`전체 음성 재생 중... (${audioChunks.length}/${chunks.length}개 부분 결합)`, 'success');

      audio.onended = () => {
        showStatus('긴 텍스트 음성 재생 완료!', 'success');
        setIsPlaying(false);
        setCurrentAudio(null);
        URL.revokeObjectURL(audioUrl);
      };

      audio.onerror = (e) => {
        console.error('오디오 재생 오류:', e);
        showStatus('오디오 재생 중 오류 발생', 'error');
        setIsPlaying(false);
        setCurrentAudio(null);
        URL.revokeObjectURL(audioUrl);
      };

    } catch (error) {
      console.error('긴 텍스트 처리 전체 오류:', error);
      showStatus('긴 텍스트 처리 오류: ' + (error as Error).message, 'error');
    }

    try {
      const response = await fetch(`https://${region}.api.cognitive.microsoft.com/sts/v1.0/issueToken`, {
        method: 'POST',
        headers: {
          'Ocp-Apim-Subscription-Key': apiKey,
          'Content-Type': 'application/x-www-form-urlencoded'
        }
      });

      if (response.ok) {
        showStatus('API 연결 성공!', 'success');
      } else {
        showStatus('API 연결 실패: ' + response.status, 'error');
      }
    } catch (error) {
      showStatus('연결 오류: ' + (error as Error).message, 'error');
    }
  };

  const splitTextIntoChunks = (text: string, maxLength: number = 2000): string[] => {
    // 먼저 문단으로 나누기
    const paragraphs = text.split(/\n\s*\n/).filter(p => p.trim());
    const chunks: string[] = [];
    
    for (const paragraph of paragraphs) {
      if (paragraph.length <= maxLength) {
        chunks.push(paragraph.trim());
      } else {
        // 문단이 너무 길면 문장으로 나누기
        const sentences = paragraph.split(/(?<=[.!?])\s+/).filter(s => s.trim());
        let currentChunk = '';
        
        for (const sentence of sentences) {
          const testChunk = currentChunk + (currentChunk ? ' ' : '') + sentence;
          
          if (testChunk.length <= maxLength) {
            currentChunk = testChunk;
          } else {
            if (currentChunk) {
              chunks.push(currentChunk.trim());
              currentChunk = sentence;
            } else {
              // 문장도 너무 길면 강제로 자르기
              const words = sentence.split(' ');
              let wordChunk = '';
              
              for (const word of words) {
                if ((wordChunk + ' ' + word).length <= maxLength) {
                  wordChunk += (wordChunk ? ' ' : '') + word;
                } else {
                  if (wordChunk) chunks.push(wordChunk.trim());
                  wordChunk = word;
                }
              }
              if (wordChunk) chunks.push(wordChunk.trim());
            }
          }
        }
        
        if (currentChunk) {
          chunks.push(currentChunk.trim());
        }
      }
    }
    
    return chunks.filter(chunk => chunk.length > 0);
  };

  const speakText = async () => {
    if (!apiKey || !region) {
      showStatus('API Key와 Region을 입력해주세요.', 'error');
      return;
    }

    if (!text.trim()) {
      showStatus('텍스트를 입력해주세요.', 'error');
      return;
    }

    // 긴 텍스트인지 확인 (임계값을 낮춤)
    if (text.length > 2000) {
      await speakLongText();
      return;
    }

    showStatus('음성 생성 중...', 'loading');

    try {
      // 1. 액세스 토큰 발급
      const tokenResponse = await fetch(`https://${region}.api.cognitive.microsoft.com/sts/v1.0/issueToken`, {
        method: 'POST',
        headers: {
          'Ocp-Apim-Subscription-Key': apiKey
        }
      });

      if (!tokenResponse.ok) {
        throw new Error('토큰 발급 실패');
      }

      const accessToken = await tokenResponse.text();

      // 2. SSML 생성
      const ssml = `
        <speak version='1.0' xml:lang='en-US'>
          <voice xml:lang='en-US' name='${voice}'>
            <prosody rate='${rate}'>
              ${text}
            </prosody>
          </voice>
        </speak>
      `;

      // 3. TTS API 호출
      const ttsResponse = await fetch(`https://${region}.tts.speech.microsoft.com/cognitiveservices/v1`, {
        method: 'POST',
        headers: {
          'Authorization': 'Bearer ' + accessToken,
          'Content-Type': 'application/ssml+xml',
          'X-Microsoft-OutputFormat': 'audio-16khz-128kbitrate-mono-mp3'
        },
        body: ssml
      });

      if (!ttsResponse.ok) {
        throw new Error('TTS 요청 실패: ' + ttsResponse.status);
      }

      // 4. 오디오 재생
      const audioBlob = await ttsResponse.blob();
      setLastAudioBlob(audioBlob);

      const audioUrl = URL.createObjectURL(audioBlob);
      const audio = new Audio(audioUrl);

      audio.play();
      showStatus('음성 재생 중... (MP3 다운로드 가능)', 'success');

      audio.onended = () => {
        showStatus('음성 재생 완료!', 'success');
        URL.revokeObjectURL(audioUrl);
      };

    } catch (error) {
      showStatus('오류: ' + (error as Error).message, 'error');
    }
  };

  const stopAudio = () => {
    if (currentAudio) {
      currentAudio.pause();
      currentAudio.currentTime = 0;
      setCurrentAudio(null);
      setIsPlaying(false);
      showStatus('음성 재생이 중지되었습니다.', 'success');
    }
  };

  const generateMp3Only = async () => {
    if (!apiKey || !region) {
      showStatus('API Key와 Region을 입력해주세요.', 'error');
      return;
    }

    if (!text.trim()) {
      showStatus('텍스트를 입력해주세요.', 'error');
      return;
    }

    if (text.length > 2000) {
      await generateLongMp3();
      return;
    }

    showStatus('MP3 파일 생성 중...', 'loading');

    try {
      const tokenResponse = await fetch(`https://${region}.api.cognitive.microsoft.com/sts/v1.0/issueToken`, {
        method: 'POST',
        headers: {
          'Ocp-Apim-Subscription-Key': apiKey
        }
      });

      if (!tokenResponse.ok) {
        throw new Error('토큰 발급 실패');
      }

      const accessToken = await tokenResponse.text();
      const escapedText = text
        .replace(/&/g, '&amp;')
        .replace(/</g, '&lt;')
        .replace(/>/g, '&gt;')
        .replace(/"/g, '&quot;')
        .replace(/'/g, '&apos;');

      const ssml = `<speak version='1.0' xml:lang='en-US'><voice xml:lang='en-US' name='${voice}'><prosody rate='${rate}'>${escapedText}</prosody></voice></speak>`;

      const ttsResponse = await fetch(`https://${region}.tts.speech.microsoft.com/cognitiveservices/v1`, {
        method: 'POST',
        headers: {
          'Authorization': 'Bearer ' + accessToken,
          'Content-Type': 'application/ssml+xml',
          'X-Microsoft-OutputFormat': 'audio-16khz-128kbitrate-mono-mp3'
        },
        body: ssml
      });

      if (!ttsResponse.ok) {
        throw new Error('TTS 요청 실패: ' + ttsResponse.status);
      }

      const audioBlob = await ttsResponse.blob();
      setLastAudioBlob(audioBlob);
      
      // 자동으로 다운로드
      downloadMp3(audioBlob);

    } catch (error) {
      showStatus('MP3 생성 오류: ' + (error as Error).message, 'error');
    }
  };

  const generateLongMp3 = async () => {
    const chunks = splitTextIntoChunks(text);
    
    if (chunks.length === 0) {
      showStatus('텍스트 분할 실패', 'error');
      return;
    }
    
    showStatus(`MP3 파일 생성 중... (${chunks.length}개 부분)`, 'loading');
    const audioChunks: Blob[] = [];
    let accessToken = '';

    try {
      const tokenResponse = await fetch(`https://${region}.api.cognitive.microsoft.com/sts/v1.0/issueToken`, {
        method: 'POST',
        headers: {
          'Ocp-Apim-Subscription-Key': apiKey
        }
      });

      if (!tokenResponse.ok) {
        throw new Error(`토큰 발급 실패: ${tokenResponse.status}`);
      }

      accessToken = await tokenResponse.text();

      for (let i = 0; i < chunks.length; i++) {
        showStatus(`MP3 생성 중... (${i + 1}/${chunks.length})`, 'loading');

        const escapedText = chunks[i]
          .replace(/&/g, '&amp;')
          .replace(/</g, '&lt;')
          .replace(/>/g, '&gt;')
          .replace(/"/g, '&quot;')
          .replace(/'/g, '&apos;');

        const ssml = `<speak version='1.0' xml:lang='en-US'><voice xml:lang='en-US' name='${voice}'><prosody rate='${rate}'>${escapedText}</prosody></voice></speak>`;

        const ttsResponse = await fetch(`https://${region}.tts.speech.microsoft.com/cognitiveservices/v1`, {
          method: 'POST',
          headers: {
            'Authorization': 'Bearer ' + accessToken,
            'Content-Type': 'application/ssml+xml',
            'X-Microsoft-OutputFormat': 'audio-16khz-128kbitrate-mono-mp3'
          },
          body: ssml
        });

        if (!ttsResponse.ok) {
          if (ttsResponse.status === 401) {
            const newTokenResponse = await fetch(`https://${region}.api.cognitive.microsoft.com/sts/v1.0/issueToken`, {
              method: 'POST',
              headers: {
                'Ocp-Apim-Subscription-Key': apiKey
              }
            });
            
            if (newTokenResponse.ok) {
              accessToken = await newTokenResponse.text();
              i--;
              continue;
            }
          }
          throw new Error(`TTS 요청 실패 (청크 ${i + 1}): ${ttsResponse.status}`);
        }

        const audioBlob = await ttsResponse.blob();
        if (audioBlob.size > 0) {
          audioChunks.push(audioBlob);
        }

        if (i < chunks.length - 1) {
          await new Promise(resolve => setTimeout(resolve, 1500));
        }
      }

      if (audioChunks.length === 0) {
        throw new Error('처리된 오디오 청크가 없습니다');
      }

      const combinedBlob = new Blob(audioChunks, { type: 'audio/mpeg' });
      setLastAudioBlob(combinedBlob);
      
      // 자동으로 다운로드
      downloadMp3(combinedBlob);

    } catch (error) {
      showStatus('긴 텍스트 MP3 생성 오류: ' + (error as Error).message, 'error');
    }
  };
  const downloadMp3 = (blob?: Blob) => {
    const audioBlob = blob || lastAudioBlob;
    
    if (!audioBlob) {
      showStatus('먼저 음성을 생성해주세요.', 'error');
      return;
    }

    const url = URL.createObjectURL(audioBlob);
    const a = document.createElement('a');
    a.href = url;
    a.download = 'azure-speech-' + new Date().getTime() + '.mp3';
    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
    URL.revokeObjectURL(url);

    showStatus('MP3 파일 다운로드 완료!', 'success');
  };

  const getStatusStyle = (): React.CSSProperties => {
    const baseStyle: React.CSSProperties = {
      marginTop: '20px',
      padding: '10px',
      border: '1px solid #ccc',
      display: status.message ? 'block' : 'none'
    };

    if (status.type === 'error') {
      return {
        ...baseStyle,
        backgroundColor: '#ffebee',
        color: '#c62828',
        borderColor: '#e57373'
      };
    } else if (status.type === 'success') {
      return {
        ...baseStyle,
        backgroundColor: '#e8f5e8',
        color: '#2e7d32',
        borderColor: '#81c784'
      };
    } else {
      return {
        ...baseStyle,
        backgroundColor: '#fff3e0',
        color: '#ef6c00',
        borderColor: '#ffb74d'
      };
    }
  };

  return (
    <div style={{ maxWidth: '800px', margin: '0 auto', padding: '20px' }}>
      <h1>Azure Speech Text-to-Speech</h1>

      <h3>API 설정</h3>
      <input
        type="text"
        value={apiKey}
        onChange={(e) => setApiKey(e.target.value)}
        placeholder="Azure Speech API Key"
        style={{ width: '300px', marginRight: '10px' }}
      />
      <input
        type="text"
        value={region}
        onChange={(e) => setRegion(e.target.value)}
        placeholder="Region (예: koreacentral)"
        style={{ width: '200px', marginRight: '10px' }}
      />
      <button onClick={testConnection}>연결 테스트</button>

      <h3>음성 설정</h3>
      <select
        value={voice}
        onChange={(e) => setVoice(e.target.value)}
        style={{ width: '250px', marginRight: '10px' }}
      >
        <option value="en-US-AvaNeural">en-US-AvaNeural (여성)</option>
        <option value="en-US-AndrewNeural">en-US-AndrewNeural (남성)</option>
        <option value="en-US-EmmaNeural">en-US-EmmaNeural (여성)</option>
        <option value="en-US-BrianNeural">en-US-BrianNeural (남성)</option>
        <option value="en-GB-SoniaNeural">en-GB-SoniaNeural (영국 여성)</option>
        <option value="en-GB-RyanNeural">en-GB-RyanNeural (영국 남성)</option>
      </select>

      속도: 
      <select
        value={rate}
        onChange={(e) => setRate(e.target.value)}
        style={{ marginLeft: '10px' }}
      >
        <option value="x-slow">매우 느리게</option>
        <option value="slow">느리게</option>
        <option value="medium">보통</option>
        <option value="fast">빠르게</option>
        <option value="x-fast">매우 빠르게</option>
      </select>

      <h3>텍스트</h3>
      <textarea
        value={text}
        onChange={(e) => setText(e.target.value)}
        rows={8}
        cols={60}
        placeholder="여기에 영어 텍스트를 입력하세요... (긴 텍스트도 자동으로 분할 처리됩니다)"
        style={{ width: '100%', maxWidth: '600px' }}
      />
      <div style={{ fontSize: '12px', color: '#666', marginTop: '5px' }}>
        현재 글자 수: {text.length} / 권장: 2,000자 이하 (긴 텍스트는 자동 분할됩니다)
      </div>
      <br />

      <button 
        onClick={speakText} 
        disabled={isPlaying}
        style={{ 
          marginRight: '10px',
          opacity: isPlaying ? 0.6 : 1,
          cursor: isPlaying ? 'not-allowed' : 'pointer'
        }}
      >
        🔊 말하기
      </button>
      
      {isPlaying && (
        <button 
          onClick={stopAudio}
          style={{ 
            marginRight: '10px',
            backgroundColor: '#dc3545',
            color: 'white',
            border: 'none',
            padding: '12px 20px',
            borderRadius: '8px',
            cursor: 'pointer'
          }}
        >
          ⏹️ 정지
        </button>
      )}
      
      <button 
        onClick={generateMp3Only}
        style={{ 
          marginRight: '10px',
          backgroundColor: '#28a745',
          color: 'white',
          border: 'none',
          padding: '12px 20px',
          borderRadius: '8px',
          cursor: 'pointer'
        }}
      >
        💾 MP3 생성/다운로드
      </button>
      
      {lastAudioBlob && (
        <button 
          onClick={() => downloadMp3()}
          style={{ 
            backgroundColor: '#17a2b8',
            color: 'white',
            border: 'none',
            padding: '12px 20px',
            borderRadius: '8px',
            cursor: 'pointer'
          }}
        >
          📁 마지막 MP3 다시 다운로드
        </button>
      )}

      <div style={getStatusStyle()}>
        {status.message}
      </div>
    </div>
  );
}

---

## 📌 Navigation
- [← All Posts](/posts)
- [🏠 Home](/)
<!-- - [📧 Contact](/contact) -->

{% if page.previous %}
**Previous:** [{{ page.previous.title }}]({{ page.previous.url }})
{% endif %}

{% if page.next %}
**Next:** [{{ page.next.title }}]({{ page.next.url }})
{% endif %}
