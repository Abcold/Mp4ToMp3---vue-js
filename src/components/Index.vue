<template>
  <div>
    <header>
      <div class="explain">
        <h1>在线免费MP4 视频转MP3</h1>
        <!-- <h5>学习弯道超车技术asd</h5> -->
      </div>
    </header>
    <div class="index-wrapper">
      <ul>
        <!-- <li class="blog-wrapper" @click="$parent.location('/details')"> -->
        <li class="blog-wrapper">
          <!-- <p class="blog-time">2018年07月30日</p>
          <h3 class="blog-title">基于 MIME 类型的服务端推送123</h3> -->
          <div class="blog-content">
            <!-- <input
              type="file"
              onchange="document.querySelector('button').removeAttribute('disabled');"
            /><br />
            <button @click="go()" disabled>Create AudioBuffer</button><br />
            <button disabled>Play/Download WAV</button> -->
            <iframe src="../../static/mp4Tomp3/JS音频转换器 -- 在浏览器中转换你的MP3，WAV，OGG，AAC，WMA文件.html" scrolling="yes" style="width: 100%;height: 1200px;" frameborder="0"></iframe>
           <!-- <span class="blog-more">阅读原文...</span> -->
          </div>
          <!-- <div class="blog-tag">
            <ul>
              <li>文章标签</li>
              <li>文章标签</li>
              <li>文章标签</li>
            </ul>
          </div> -->
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
export default {
  name: "index",
  data() {
    return {
    };
  },
  methods: {
    go() {
      // this.isshowTagModel = !this.isshowTagModel;
      document.querySelector("button").setAttribute("disabled", "");
      document.querySelectorAll("button")[1].setAttribute("disabled", "");
      var audioContext = new (window.AudioContext ||
          window.webkitAudioContext)(),
        reader = new FileReader(),
        myBuffer;
      reader.onload = function() {
        document.querySelector("button").removeAttribute("disabled");
        var videoFileAsBuffer = reader.result;
        audioContext.decodeAudioData(videoFileAsBuffer, function(
          decodedAudioData
        ) {
          try {
            var duration = decodedAudioData.duration;
            var offlineAudioContext = new (window.OfflineAudioContext ||
              window.webkitOfflineAudioContext)(2, 44100 * duration, 44100);
            var soundSource = offlineAudioContext.createBufferSource();
            myBuffer = decodedAudioData;
            soundSource.buffer = myBuffer;
            soundSource.connect(offlineAudioContext.destination);
            var source = audioContext.createBufferSource();
            offlineAudioContext.oncomplete = function(e) {
              document.querySelectorAll("button")[1].onclick = function() {
                this.setAttribute("disabled", "");
                source.buffer = e.renderedBuffer;
                source.connect(audioContext.destination);
                source.start ? source.start(0) : source.noteOn(0);
                exportWAVSampleAndSave(e.renderedBuffer, function(blob) {
                  window.open(window.URL.createObjectURL(blob), "");
                });
              };
              document
                .querySelectorAll("button")[1]
                .removeAttribute("disabled");
            };
            offlineAudioContext.startRendering();
            soundSource.start ? soundSource.start(0) : soundSource.noteOn(0);
          } catch (ex) {
            alert(ex);
          }
        });
      };
      reader.readAsArrayBuffer(
        document.querySelector("input[type=file]").files[0]
      );
    },
    interleave(chanData){
    var chans  = chanData.length;
    if(chans === 1) return chanData[0];
    var l0     = chanData[0].length;
    var length = l0 * chans;
    var result = new Float32Array(length);
    // --
    var index = 0;
    var inputIndex = 0;
    for(var i=0; i<l0; i++){for(var c=0; c<chans; c++){result[i*chans+c] = chanData[c][i];}}
    console.log(result);
    return result;
  },
  floatTo16BitPCM(output, offset, input){try{
    for (var i = 0; i < input.length; i++, offset+=2){
      var s = Math.max(-1, Math.min(1, input[i]));
      output.setInt16(offset, s < 0 ? s * 0x8000 : s * 0x7FFF, true);
    }
  }catch(ex){alert(ex);}},
  wr(view, offset, string){for (var i = 0; i < string.length; i++){view.setUint8(offset + i, string.charCodeAt(i));}} ,
   getAudioSamplesAsWavBlob(samples, interleavedChannels, sampleRate){
    // see: https://ccrma.stanford.edu/courses/422/projects/WaveFormat/
    interleavedChannels = interleavedChannels||1;
    console.log("encoding wav: samples:"+samples.length+", chans:"+interleavedChannels+", rate:"+sampleRate);
    var buffer  = new ArrayBuffer(44 + samples.length * 2); // 44 + PCM points * 2
    var dv      = new DataView(buffer);
    // -- header
    wr(dv, 0, 'RIFF');   // RIFF
    dv.setUint32(4, 32 + samples.length * interleavedChannels, true); // 32 + length
    wr(dv, 8, 'WAVE');   // RIFF type
    // -- chunk 1
    wr(dv, 12, 'fmt ');  // chunk id
    dv.setUint32(16, 16, true);   // subchunk1size (16 for PCM)
    dv.setUint16(20, 1, true);    // 1=PCM
    dv.setUint16(22, interleavedChannels, true); // num channels
    dv.setUint32(24, sampleRate, true);          // samplerate
    dv.setUint32(28, sampleRate * interleavedChannels * 2, true); // byterate
    dv.setUint16(32, 2 * interleavedChannels, true);  // block align
    dv.setUint16(34, 16, true); // bits per sample (16 = 2 bytes)
    // -- chunk 2
    wr(dv, 36, 'data');         // data chunk id
    dv.setUint32(40, samples.length * interleavedChannels, true); // chunk len
    floatTo16BitPCM(dv, 44, samples);
    var wavBlob = new Blob([dv], {type: "audio/wav"});
    return wavBlob;
  },
    exportWAVSampleAndSave(sample, cb){
    var chanData = [];
    var chans    = sample.numberOfChannels;
    console.log("Sample channels:", chans);
    for(var c=0; c<chans; c++){chanData.push(sample.getChannelData(c));}
    var sample_chandata = interleave(chanData);
    console.log("interleaved chandata length:", sample_chandata.length);
    // --
    return cb(getAudioSamplesAsWavBlob(sample_chandata, chans, sample.sampleRate));
  },

  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.index-wrapper {
  max-width: 90%;
  margin: 30px auto 40px;
}
.blog-wrapper {
  margin-bottom: 30px;
  padding: 12px 12px 0;
  background: #fff;
  border-radius: 3px;
  box-shadow: 0 1px 2px rgba(151, 151, 151, 0.58);
}
.blog-time {
  line-height: 24px;
  margin: 0 0 10px;
  font-size: 13px;
  font-weight: bold;
  color: #727272;
  overflow: hidden;
}
.blog-title {
  margin-bottom: 15px;
  font-size: 24px;
  line-height: 32px;
  color: #3f51b5;
}
.blog-content {
  word-break: break-all;
  padding-bottom: 20px;
  line-height: 1.8;
}
.blog-more {
  display: inline-block;
  padding: 0 6px;
  font-weight: 500;
  color: #3f51b5 !important;
  border: none !important;
  border-radius: 3px;
}
.blog-tag {
  position: relative;
  margin: 0 -12px;
  padding: 12px 20px 8px;
  border-top: 1px solid #ddd;
}
.blog-tag li {
  display: inline-block;
  margin: 0 8px 8px 0;
  border-radius: 2px;
  background: #8bc34a;
  padding: 0 16px;
  line-height: 28px;
  color: rgba(255, 255, 255, 0.8);
}
</style>
