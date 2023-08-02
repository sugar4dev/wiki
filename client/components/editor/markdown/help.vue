<template>  
  <v-card class="editor-markdown-help animated fadeInLeft" flat tile>
    <v-container class="pa-3" grid-list-lg fluid>
      <v-layout row wrap>
        <v-flex xs12 lg6 xl4>
          <v-card class="radius-7 animated fadeInUp" light>
            <v-card-text>
              <div class="d-flex">
                <v-toolbar class="radius-7" color="teal lighten-5" dense flat height="44">
                  <v-icon class="mr-3" color="teal">mdi-information-variant</v-icon>
                  <div class="body-2 teal--text">Markdown Reference</div>
                </v-toolbar>
              </div>
              <div class="body-2 mt-3">Bold</div>
              <v-layout row>
                <v-flex xs6>
                  <v-card class="editor-markdown-help-source" flat>
                    <v-card-text>
                      <div>**Lorem ipsum**</div>
                    </v-card-text>
                  </v-card>
                </v-flex>
                <v-icon>mdi-chevron-right</v-icon>
                <v-flex>
                  <v-card class="editor-markdown-help-result" flat>
                    <v-card-text>
                      <div class="caption"><strong>Lorem ipsum</strong></div>
                    </v-card-text>
                  </v-card>
                </v-flex>
              </v-layout>
              <div class="body-2 mt-3">Italic</div>
              <v-layout row>
                <v-flex xs6>
                  <v-card class="editor-markdown-help-source" flat>
                    <v-card-text>
                      <div>*Lorem ipsum*</div>
                    </v-card-text>
                  </v-card>
                </v-flex>
                <v-icon>mdi-chevron-right</v-icon>
                <v-flex>
                  <v-card class="editor-markdown-help-result" flat>
                    <v-card-text>
                      <div class="caption"><em>Lorem ipsum</em></div>
                    </v-card-text>
                  </v-card>
                </v-flex>
              </v-layout>
              <div class="body-2 mt-3">Strikethrough</div>
              <v-layout row>
                <v-flex xs6>
                  <v-card class="editor-markdown-help-source" flat>
                    <v-card-text>
                      <div>~~Lorem ipsum~~</div>
                    </v-card-text>
                  </v-card>
                </v-flex>
                <v-icon>mdi-chevron-right</v-icon>
                <v-flex>
                  <v-card class="editor-markdown-help-result" flat>
                    <v-card-text>
                      <div class="caption" style="text-decoration: line-through;">Lorem ipsum</div>
                    </v-card-text>
                  </v-card>
                </v-flex>
              </v-layout>
              <div class="body-2 mt-3">Headers</div>
              <v-layout row>
                <v-flex xs6>
                  <v-card class="editor-markdown-help-source" flat>
                    <v-card-text>
                      <div># Header 1</div>
                      <div>## Header 2</div>
                      <div>### Header 3</div>
                      <div>#### Header 4</div>
                      <div>##### Header 5</div>
                      <div>###### Header 6</div>
                    </v-card-text>
                  </v-card>
                </v-flex>
                <v-icon>mdi-chevron-right</v-icon>
                <v-flex>
                  <v-card class="editor-markdown-help-result" flat>
                    <v-card-text>
                      <div style="font-weight: 700; font-size: 24px;">Header 1</div>
                      <div style="font-weight: 700; font-size: 22px;">Header 2</div>
                      <div style="font-weight: 700; font-size: 20px;">Header 3</div>
                      <div style="font-weight: 700; font-size: 18px;">Header 4</div>
                      <div style="font-weight: 700; font-size: 16px;">Header 5</div>
                      <div style="font-weight: 700; font-size: 14px;">Header 6</div>
                    </v-card-text>
                  </v-card>
                </v-flex>
              </v-layout>
              <div class="body-2 mt-3">Unordered Lists</div>
              <div class="caption grey--text text--darken-1"><em>You can also use the asterisk symbol instead of the dash.</em></div>
              <v-layout row>
                <v-flex xs6>
                  <v-card class="editor-markdown-help-source" flat>
                    <v-card-text>
                      <div>- Unordered List Item 1</div>
                      <div>- Unordered List Item 2</div>
                      <div>- Unordered List Item 3</div>
                    </v-card-text>
                  </v-card>
                </v-flex>
                <v-icon>mdi-chevron-right</v-icon>
                <v-flex>
                  <v-card class="editor-markdown-help-result" flat>
                    <v-card-text>
                      <ul>
                        <li>Unordered List Item 1</li>
                        <li>Unordered List Item 2</li>
                        <li>Unordered List Item 3</li>
                      </ul>
                    </v-card-text>
                  </v-card>
                </v-flex>
              </v-layout>
              <div class="body-2 mt-3">Ordered Lists</div>
              <div class="caption grey--text text--darken-1"><em>Even though we prefix all lines with <strong>1.</strong>, the output will be correctly numbered automatically.</em></div>
              <v-layout row>
                <v-flex xs6>
                  <v-card class="editor-markdown-help-source" flat>
                    <v-card-text>
                      <div>1. Ordered List Item 1</div>
                      <div>1. Ordered List Item 2</div>
                      <div>1. Ordered List Item 3</div>
                    </v-card-text>
                  </v-card>
                </v-flex>
                <v-icon>mdi-chevron-right</v-icon>
                <v-flex>
                  <v-card class="editor-markdown-help-result" flat>
                    <v-card-text>
                      <ol>
                        <li>Unordered List Item 1</li>
                        <li>Unordered List Item 2</li>
                        <li>Unordered List Item 3</li>
                      </ol>
                    </v-card-text>
                  </v-card>
                </v-flex>
              </v-layout>
              <div class="body-2 mt-3">Images</div>
              <v-layout row>
                <v-flex xs6>
                  <v-card class="editor-markdown-help-source" flat>
                    <v-card-text>
                      <div>![Caption Text](/path/to/image.jpg)</div>
                    </v-card-text>
                  </v-card>
                </v-flex>
                <v-icon>mdi-chevron-right</v-icon>
                <v-flex>
                  <v-card class="editor-markdown-help-result" flat>
                    <v-card-text><img src="https://via.placeholder.com/150x50.png"></v-card-text>
                  </v-card>
                </v-flex>
              </v-layout>
            </v-card-text>
          </v-card>
        </v-flex>
        <v-flex xs12 lg6 xl4>
          <v-card class="radius-7 animated fadeInUp wait-p1s" light>
            <v-card-text>
              <div class="d-flex">
                <v-toolbar class="radius-7" color="teal lighten-5" dense flat height="44">
                  <v-icon class="mr-3" color="teal">mdi-information-variant</v-icon>
                  <div class="body-2 teal--text">Markdown Reference (continued)</div>
                </v-toolbar>
              </div>
              <div class="body-2 mt-3">Links</div>
              <v-layout row>
                <v-flex xs6>
                  <v-card class="editor-markdown-help-source" flat>
                    <v-card-text>
                      <div>[Link Text](https://wiki.js.org)</div>
                    </v-card-text>
                  </v-card>
                </v-flex>
                <v-icon>mdi-chevron-right</v-icon>
                <v-flex>
                  <v-card class="editor-markdown-help-result" flat>
                    <v-card-text>
                      <div class="caption"><a href="https://wiki.js.org" target="_blank">Link Text</a></div>
                    </v-card-text>
                  </v-card>
                </v-flex>
              </v-layout>
              <div class="body-2 mt-3">Superscript</div>
              <v-layout row>
                <v-flex xs6>
                  <v-card class="editor-markdown-help-source" flat>
                    <v-card-text>
                      <div>Lorem ^ipsum^</div>
                    </v-card-text>
                  </v-card>
                </v-flex>
                <v-icon>mdi-chevron-right</v-icon>
                <v-flex>
                  <v-card class="editor-markdown-help-result" flat>
                    <v-card-text>
                      <div class="caption">Lorem <sup>ipsum</sup></div>
                    </v-card-text>
                  </v-card>
                </v-flex>
              </v-layout>
              <div class="body-2 mt-3">Subscript</div>
              <v-layout row>
                <v-flex xs6>
                  <v-card class="editor-markdown-help-source" flat>
                    <v-card-text>
                      <div>Lorem ~ipsum~</div>
                    </v-card-text>
                  </v-card>
                </v-flex>
                <v-icon>mdi-chevron-right</v-icon>
                <v-flex>
                  <v-card class="editor-markdown-help-result" flat>
                    <v-card-text>
                      <div class="caption"><em>Lorem <sub>ipsum</sub></em></div>
                    </v-card-text>
                  </v-card>
                </v-flex>
              </v-layout>
              <div class="body-2 mt-3">Horizontal Line</div>
              <v-layout row>
                <v-flex xs6>
                  <v-card class="editor-markdown-help-source" flat>
                    <v-card-text>
                      <div>Lorem ipsum</div>
                      <div>---</div>
                      <div>Dolor sit amet</div>
                    </v-card-text>
                  </v-card>
                </v-flex>
                <v-icon>mdi-chevron-right</v-icon>
                <v-flex>
                  <v-card class="editor-markdown-help-result" flat>
                    <v-card-text>
                      <div class="caption">Lorem ipsum</div>
                      <v-divider class="my-2"></v-divider>
                      <div class="caption">Dolor sit amet</div>
                    </v-card-text>
                  </v-card>
                </v-flex>
              </v-layout>
              <div class="body-2 mt-3">Inline Code</div>
              <v-layout row>
                <v-flex xs6>
                  <v-card class="editor-markdown-help-source" flat>
                    <v-card-text>
                      <div>Lorem `ipsum dolor sit` amet</div>
                    </v-card-text>
                  </v-card>
                </v-flex>
                <v-icon>mdi-chevron-right</v-icon>
                <v-flex>
                  <v-card class="editor-markdown-help-result" flat>
                    <v-card-text>
                      <div class="caption">Lorem <code>ipsum dolor sit</code> amet</div>
                    </v-card-text>
                  </v-card>
                </v-flex>
              </v-layout>
              <div class="body-2 mt-3">Code Blocks</div>
              <div class="caption grey--text text--darken-1"><em>In the example below, <strong>js</strong> defines the syntax highlighting language to use. It can be omitted.</em></div>
              <v-layout row>
                <v-flex xs6>
                  <v-card class="editor-markdown-help-source" flat>
                    <v-card-text>
                      <div>```js</div>
                      <div>function main () {</div>
                      <div class="pl-3">echo 'Lorem ipsum'</div>
                      <div>}</div>
                      <div>```</div>
                    </v-card-text>
                  </v-card>
                </v-flex>
                <v-icon>mdi-chevron-right</v-icon>
                <v-flex>
                  <v-card class="editor-markdown-help-result" flat>
                    <v-card-text class="contents">
                      <pre class="prismjs line-numbers language-js"><code class="language-js"><span class="token keyword">function</span><span class="token function"> main</span><span class="token punctuation"> (</span><span class="token punctuation">)</span><span class="token punctuation"> {<br></span>  echo<span class="token string"> 'Lorem ipsum'<br></span><span class="token punctuation">}</span><span class="line-numbers-rows" aria-hidden="true"><span></span><span></span><span></span></span></code></pre>
                    </v-card-text>
                  </v-card>
                </v-flex>
              </v-layout>
              <div class="body-2 mt-3">Blockquotes</div>
              <v-layout row>
                <v-flex xs6>
                  <v-card class="editor-markdown-help-source" flat>
                    <v-card-text>
                      <div>&gt; Lorem ipsum</div>
                      <div>&gt; dolor sit amet</div>
                      <div>&gt; consectetur adipiscing elit</div>
                    </v-card-text>
                  </v-card>
                </v-flex>
                <v-icon>mdi-chevron-right</v-icon>
                <v-flex>
                  <v-card class="editor-markdown-help-result" flat>
                    <v-card-text>
                      <blockquote style="border: 1px solid #263238; border-radius: .5rem; padding: 1rem 24px;">Lorem ipsum<br>dolor sit amet<br>consectetur adipiscing elit</blockquote>
                    </v-card-text>
                  </v-card>
                </v-flex>
              </v-layout>
            </v-card-text>
          </v-card>
        </v-flex>
        <v-flex xs12 xl4>
          <v-card class="radius-7 animated fadeInUp wait-p2s" light>
            <v-card-text>
              <v-toolbar class="radius-7" color="teal lighten-5" dense flat>
                <v-icon class="mr-3" color="teal">mdi-keyboard</v-icon>
                <div class="body-2 teal--text">Keyboard Shortcuts</div>
              </v-toolbar>
              <v-list class="editor-markdown-help-kbd" two-line dense>
                <v-list-item>
                  <v-list-item-content class="body-2">Bold</v-list-item-content>
                  <v-list-item-action><kbd>{{ctrlKey}}</kbd> + <kbd>B</kbd></v-list-item-action>
                </v-list-item>
                <v-divider></v-divider>
                <v-list-item>
                  <v-list-item-content class="body-2">Italic</v-list-item-content>
                  <v-list-item-action><kbd>{{ctrlKey}}</kbd> + <kbd>I</kbd></v-list-item-action>
                </v-list-item>
                <v-divider></v-divider>
                <v-list-item>
                  <v-list-item-content class="body-2">Increase Header Level</v-list-item-content>
                  <v-list-item-action><kbd>{{ctrlKey}}</kbd> + <kbd>{{altKey}}</kbd> + <kbd>Right</kbd></v-list-item-action>
                </v-list-item>
                <v-divider></v-divider>
                <v-list-item>
                  <v-list-item-content class="body-2">Decrease Header Level</v-list-item-content>
                  <v-list-item-action><kbd>{{ctrlKey}}</kbd> + <kbd>{{altKey}}</kbd> + <kbd>Left</kbd></v-list-item-action>
                </v-list-item>
                <v-divider></v-divider>
                <v-list-item>
                  <v-list-item-content class="body-2">Save</v-list-item-content>
                  <v-list-item-action><kbd>{{ctrlKey}}</kbd> + <kbd>S</kbd></v-list-item-action>
                </v-list-item>
                <v-divider></v-divider>
                <v-list-item>
                  <v-list-item-content class="body-2">Undo</v-list-item-content>
                  <v-list-item-action><kbd>{{ctrlKey}}</kbd> + <kbd>Z</kbd></v-list-item-action>
                </v-list-item>
                <v-divider></v-divider>
                <v-list-item>
                  <v-list-item-content class="body-2">Redo</v-list-item-content>
                  <v-list-item-action><kbd>{{ctrlKey}}</kbd> + <kbd>Y</kbd></v-list-item-action>
                </v-list-item>
                <v-divider></v-divider>
                <v-list-item>
                  <v-list-item-content>
                    <v-list-item-title class="body-2">Distraction Free Mode</v-list-item-title>
                    <v-list-item-subtitle>Press <kbd>Esc</kbd> to exit.</v-list-item-subtitle>
                  </v-list-item-content>
                  <v-list-item-action><kbd>F11</kbd></v-list-item-action>
                </v-list-item>
              </v-list>
            </v-card-text>
          </v-card>
          <v-card class="radius-7 animated fadeInUp wait-p3s mt-3" light>
            <v-card-text>
              <v-toolbar class="radius-7" color="teal lighten-5" dense flat>
                <v-icon class="mr-3" color="teal">mdi-mouse</v-icon>
                <div class="body-2 teal--text">Multi-Selection</div>
              </v-toolbar>
              <v-list class="editor-markdown-help-kbd" two-line dense>
                <v-list-item>
                  <v-list-item-content class="body-2">Multiple Cursors</v-list-item-content>
                  <v-list-item-action><kbd>{{ctrlKey}}</kbd> + Left Click</v-list-item-action>
                </v-list-item>
                <v-divider></v-divider>
                <v-list-item>
                  <v-list-item-content class="body-2">Select Region</v-list-item-content>
                  <v-list-item-action><kbd>{{ctrlKey}}</kbd> + <kbd>{{altKey}}</kbd> + Left Click</v-list-item-action>
                </v-list-item>
                <v-divider></v-divider>
                <v-list-item>
                  <v-list-item-content class="body-2">Deselect</v-list-item-content>
                  <v-list-item-action><kbd>Esc</kbd></v-list-item-action>
                </v-list-item>
              </v-list>
            </v-card-text>
          </v-card>
        </v-flex>
      </v-layout>
    </v-container>
  </v-card>
</template>

<script>
export default {
  computed: {
    ctrlKey() { return /Mac/.test(navigator.platform) ? 'Cmd' : 'Ctrl' },
    altKey() { return /Mac/.test(navigator.platform) ? 'Option' : 'Alt' }
  }
}
</script>

<style lang='scss'>
.editor-markdown-help {
  position: fixed !important;
  top: 112px;
  left: 64px;
  z-index: 10;
  width: calc(100vw - 64px - 17px);
  height: calc(100vh - 112px - 24px);
  background-color: rgba(darken(mc('grey', '900'), 3%), .9) !important;
  overflow: auto;

  &-source {
    background-color: mc('blue-grey', '900') !important;
    border-radius: 7px;
    font-family: 'Roboto Mono', monospace;
    font-size: 14px;
    color: #FFF !important;

    .v-card__text {
      color: #FFF !important;
    }
  }

  &-result {
    background-color: mc('blue-grey', '50') !important;
    border-radius: 7px;
    font-size: 14px;

    code {
      display: inline-block;
      background-color: mc('pink', '50');
      box-shadow: none;
      font-size: inherit;
    }

    .contents {
      padding-bottom: 16px;
    }

    .prismjs {
      margin: 0;
    }
  }

  &-kbd {
    .v-list-item__action {
      flex-direction: row;
      align-items: center;

      kbd {
        display: inline-block;
        border: 1px solid #ccc;
        border-radius: 4px;
        padding: 0.1em 0.5em;
        margin: 0 0.2em;
        box-shadow: 0 1px 0px rgba(0, 0, 0, 0.2), 0 0 0 2px #fff inset;
        background-color: #f7f7f7;
        color: mc('grey', '700');
        font-size: 12px;
      }
    }
  }
}
</style>
