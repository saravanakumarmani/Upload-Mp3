<-- upload MP3 -->
<template>
  <div class="upload-episode-page">
    <div class="header-section">
      <div class="header-section-heading">Upload your episode</div>
      <div class="header-section-description">Tell your listeners a bit about this episode.</div>
      <div class="upload-mp3">
        <div class="heading">
          <div class="profile-img-header">Upload MP3</div>
          <div class="drag-image">
            Drag and drop or
            <label for="file_input_id" class="link">click here to browse</label>
          </div>
        </div>
        <div class="profile-img-container">
          <div class="profile-img">
            <div v-if="!isEpisodeUploaded">
              <img v-if="previewImage === emptyImage" :src="emptyImage" class="no-image" />
              <img v-else :src="previewImage" class="profile-image" />
            </div>
            <!-- Audio player section-->
            <div v-else class="episode-card-player-section">
              <div class="player-controls">
                <!-- Audio player -->
                <div>
                  <button class="play-button" @click.prevent="toggleplay()">
                    <v-icon class="icon">mdi-{{icon}}</v-icon>
                  </button>
                </div>
                <div class="progress-bar-container">
                  <div class="progress-bar">
                    <div
                      class="progress-bar-active"
                      v-bind:style="{width: progressPercentage +'%'}"
                    ></div>
                  </div>
                  <div class="prorgess-bar-timing">
                    <div>{{convertToTimestamp(parseInt(currentTime))}}</div>
                    <div>{{convertToTimestamp(parseInt(episodeTime))}}</div>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
           <span class="hint">Placeholder for format and size restrictions</span>
      </div>
    </div>
    <div>
      <TextField
        :rules="[v => v.length > 0 || '*Required']"
        hint="Keep it short and informative e.g. Summar product lunch"
        @emitTextValue="setValue"
        label="Episode title"
      />
      <TextArea
        rows="5"
        errorMessages
        hintText="Who are you? What are you do? Who do you do it for? Why do you it?"
        :rules="[v => v.length > 0 || '*Required']"
        @emitTextValue="setValue"
        label="About this episode"
      />
    </div>
    <div>
      <Button
        :disabled="disabled"
        @buttonClicked="saveChange"
        value="contiue"
        color="primary"
        height="48px"
        width="127px"
      />
    </div>
    <input type="file" accept="audio/*" id="file_input_id" @change="uploadEpisode" />
    <audio
      preload="metadata"
      ref="audio"
      id="upload-episode"
      @timeupdate="onTimeUpdate"
      @play="play"
      @error="error"
      @ended="ended"
    />
  </div>
</template>

<script>
import NoImage from '../assets/empty-image.png';

export default {
  name: 'UploadEpisodePage',
  data() {
    return {
      previewImage: NoImage,
      currentTime: 0,
      episodeTime: 0,
      episodeTitle: '',
      episodeDescription: '',
      progressPercentage: 0,
      icon: 'play',
      emptyImage: NoImage,
      isEpisodeUploaded: false,
    };
  },
  watch: {
    // Updating the current time to update the progress bar
    currentTime(currentTime) {
      if (Math.abs(currentTime - this.$refs.audio.currentTime) > 0.5) {
        this.$refs.audio.currentTime = currentTime;
      }
      this.progressPercentage = (currentTime / this.episodeTime) * 100;
    },
  },
  computed: {
    disabled() {
      if (
        this.previewImage
        && this.previewImage !== this.emptyImage
        && this.episodeTitle
        && this.episodeDescription
      ) {
        return false;
      }
      return true;
    },
  },
  methods: {
    error() {
      // TODO
    },
    saveChange() {
      // TODO
    },
    async uploadEpisode(e) {
      const target = e.currentTarget;
      const file = target.files[0];
      console.log('file--->', file);
      if (target.files && file) {
        const reader = new FileReader();
        reader.onload = (m) => {
          document.getElementById('upload-episode').setAttribute('src', m.target.result);
        };
        await reader.readAsDataURL(file);
        this.isEpisodeUploaded = true;
        this.currentTime = 0;
        this.icon = 'play';
        document.getElementById('upload-episode').onloadedmetadata = () => {
          this.setDuration();
        };
      }
    },
    onTimeUpdate() {
      this.currentTime = this.$refs.audio.currentTime;
    },
    toggleplay() {
      // eslint-disable-next-line no-unused-expressions
      this.icon === 'play' ? this.play() : this.pause();
      this.icon = this.icon === 'play' ? 'pause' : 'play';
    },
    setDuration() {
      this.episodeTime = document.getElementById('upload-episode').duration;
    },
    play() {
      this.setDuration();
      document.getElementById('upload-episode').play();
    },
    pause() {
      document.getElementById('upload-episode').pause();
    },
    ended() {
      this.icon = 'play';
    },
    TimeFormatter(num) {
      return `0${num}`.slice(-2);
    },
    convertToTimestamp(secs) {
      let minutes = Math.floor(secs / 60);
      secs %= 60;
      const hours = Math.floor(minutes / 60);
      minutes %= 60;
      return `${this.TimeFormatter(hours)}:${this.TimeFormatter(
        minutes,
      )}:${this.TimeFormatter(secs)}`;
    },
    setValue(value, label) {
      switch (label) {
        case 'About this episode':
          this.episodeDescription = value;
          break;
        case 'Episode title':
          this.episodeTitle = value;
          break;
        default:
          break;
      }
    },
  },
};

}
</style>
