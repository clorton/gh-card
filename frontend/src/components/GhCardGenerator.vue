<template>
  <div>
    <h3>Example</h3>
    <span v-for="exampleRepoName in exampleRepoNames">
      <a :href="getGitHubRepoUrl(exampleRepoName)">
      <img :src="getImgUrl(exampleRepoName, 'svg', true)">
    </a>
    </span>
    <hr>
    <p>
      <input type="text" v-model="repoName" placeholder="user/repo" class="repo_name">
      <button @click="update()">Generate</button><br>
    </p>
    <div v-if="imageGenerated">
      <p>
        <input type="radio" v-model="imageExtension" value="svg">SVG
        <input type="radio" v-model="imageExtension" value="png">PNG
      </p>
      <p>
        <input type="checkbox" v-model="useFullName"> Full name
      </p>

      <a :href="gitHubRepoUrl">
        <img :src="imageUrl">
      </a><br>
      <h3>Image URL</h3>
      <input type="text" :value="imageUrl" size="60">
      <h3>HTML</h3>
      <textarea cols="60" rows="3">{{ embedHtml }}</textarea>
      <h3>Markdown</h3>
      <textarea cols="60" rows="3">{{ embedMarkdown }}</textarea>
      <h3><a href="https://scrapbox.io" target="_blank">Scrapbox</a></h3>
      <textarea cols="60" rows="3">{{ embedScrapbox }}</textarea>
    </div>
  </div>
</template>

<script lang="ts">
/* tslint:disable:no-console */
import { Component, Watch, Vue } from 'vue-property-decorator';
import consts from '@/constants';

console.log(`image server is ${consts.imageServerUrl}`);

@Component
export default class GhCardGenerator extends Vue {
  private exampleRepoNames = [
    'mozilla/send',
  ];

  private repoName = '';
  private imageExtension = 'svg';
  private useFullName = false;

  private imageGenerated: boolean = false;
  private gitHubRepoUrl = '';
  private imageUrl = '';
  private embedHtml     = '';
  private embedMarkdown = '';
  private embedScrapbox = '';

  private mounted() {
    window.addEventListener('keydown', (e: WindowEventMap['keydown']) => {
      // Meta + Enter
      // (Ctrl + Enter or Cmd + Enter)
      if ((e.ctrlKey || e.metaKey) && e.key === 'Enter') {
        Vue.nextTick(() => {
          this.update();
        });
      }
    });
  }

  private get cleanRepoName(): string {
    // Remove white spaces from repo name
    return this.repoName.replace(/\s/g, '');
  }

  private update() {
    this.gitHubRepoUrl = this.getGitHubRepoUrl(this.cleanRepoName);
    this.imageUrl      = this.getImgUrl(this.cleanRepoName, this.imageExtension, this.useFullName);
    this.embedHtml     = this.getEmbedHtml(this.cleanRepoName, this.imageExtension, this.useFullName);
    this.embedMarkdown = this.getEmbedMarkdown(this.cleanRepoName, this.imageExtension, this.useFullName);
    this.embedScrapbox = this.getEmbedScrapbox(this.cleanRepoName, this.imageExtension, this.useFullName);
    // NOTE: This will be never false
    this.imageGenerated = true;
  }

  private getImgUrl(repoName: string, imageExtension: string, useFullName: boolean): string {
    const query = useFullName ? '?fullname' : '';
    return `${consts.imageServerUrl}/repos/${repoName}.${imageExtension}${query}`;
  }

  private getGitHubRepoUrl(repoName: string): string {
    return `https://github.com/${repoName}`;
  }

  private getEmbedHtml(repoName: string, imageExtension: string, useFullName: boolean): string {
    /* tslint:disable:max-line-length */
    return `<a href="${this.getGitHubRepoUrl(repoName)}"><img src="${this.getImgUrl(repoName, imageExtension, useFullName)}"></a>`;
  }

  private getEmbedMarkdown(repoName: string, imageExtension: string, useFullName: boolean): string {
    return `[![${repoName} - GitHub](${this.getImgUrl(repoName, imageExtension, useFullName)})](${this.getGitHubRepoUrl(repoName)})`;
  }

  private getEmbedScrapbox(repoName: string, imageExtension: string, useFullName: boolean): string {
    return `[${this.getImgUrl(repoName, imageExtension, useFullName)} ${this.getGitHubRepoUrl(repoName)}]`;
  }

  @Watch('imageExtension')
  private onImageExtension() {
    if (this.imageGenerated) {
      this.update();
    }
  }

  @Watch('useFullName')
  private onUseFullName() {
    if (this.imageGenerated) {
      this.update();
    }
  }

}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 1em 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
.repo_name {
  font-size: 2em;
  width: 15em;
  text-align: center;
}
button {
  margin: 1em 0;
  font-size: 2em;
}
img {
  width: 500px;
}
</style>
