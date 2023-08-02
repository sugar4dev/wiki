<template>  
  <div v-intersect.once="onIntersect">
    <v-textarea id="discussion-new" outlined flat :placeholder="$t(`common:comments.newPlaceholder`)" auto-grow dense rows="3" hide-details v-model="newcomment" color="blue-grey darken-2" :background-color="$vuetify.theme.dark ? `grey darken-5` : `white`" v-if="permissions.write" :aria-label="$t(`common:comments.fieldContent`)"></v-textarea>
    <v-row class="mt-2" dense v-if="!isAuthenticated && permissions.write">
      <v-col cols="12" lg="6">
        <v-text-field outlined color="blue-grey darken-2" :background-color="$vuetify.theme.dark ? `grey darken-5` : `white`" :placeholder="$t(`common:comments.fieldName`)" hide-details dense autocomplete="name" v-model="guestName" :aria-label="$t(`common:comments.fieldName`)"></v-text-field>
      </v-col>
      <v-col cols="12" lg="6">
        <v-text-field outlined color="blue-grey darken-2" :background-color="$vuetify.theme.dark ? `grey darken-5` : `white`" :placeholder="$t(`common:comments.fieldEmail`)" hide-details type="email" dense autocomplete="email" v-model="guestEmail" :aria-label="$t(`common:comments.fieldEmail`)"></v-text-field>
      </v-col>
    </v-row>
    <div class="d-flex align-center pt-3" v-if="permissions.write">
      <v-icon class="mr-1" color="blue-grey">mdi-language-markdown-outline</v-icon>
      <div class="caption blue-grey--text">{{$t('common:comments.markdownFormat')}}</div>
      <v-spacer></v-spacer>
      <div class="caption mr-3" v-if="isAuthenticated">
        <i18next tag="span" path="common:comments.postingAs"><strong place="name">{{userDisplayName}}</strong></i18next>
      </div>
      <v-btn dark color="blue-grey darken-2" @click="postComment" depressed :aria-label="$t(`common:comments.postComment`)">
        <v-icon left>mdi-comment</v-icon><span class="text-none">{{$t('common:comments.postComment')}}</span>
      </v-btn>
    </div>
    <v-divider class="mt-3" v-if="permissions.write"></v-divider>
    <div class="pa-5 d-flex align-center justify-center" v-if="isLoading && !hasLoadedOnce">
      <v-progress-circular indeterminate size="20" width="1" color="blue-grey"></v-progress-circular>
      <div class="caption blue-grey--text pl-3"><em>{{$t('common:comments.loading')}}</em></div>
    </div>
    <v-timeline dense v-else-if="comments && comments.length > 0">
      <v-timeline-item class="comments-post" color="pink darken-4" large v-for="cm of comments" :key="`comment-` + cm.id" :id="`comment-post-id-` + cm.id">
        <template v-slot:icon>
          <v-avatar color="blue-grey"><span class="white--text title">{{cm.initials}}</span></v-avatar>
        </template>
        <v-card class="elevation-1">
          <v-card-text>
            <div class="comments-post-actions" v-if="permissions.manage && !isBusy && commentEditId === 0">
              <v-icon class="mr-3" small @click="editComment(cm)">mdi-pencil</v-icon>
              <v-icon small @click="deleteCommentConfirm(cm)">mdi-delete</v-icon>
            </div>
            <div class="comments-post-name caption"><strong>{{cm.authorName}}</strong></div>
            <div class="comments-post-date overline grey--text">{{cm.createdAt | moment('from') }} <em v-if="cm.createdAt !== cm.updatedAt">- {{$t('common:comments.modified', { reldate: $options.filters.moment(cm.updatedAt, 'from') })}}</em></div>
            <div class="comments-post-content mt-3" v-if="commentEditId !== cm.id" v-html="cm.render"></div>
            <div class="comments-post-editcontent mt-3" v-else>
              <v-textarea outlined flat auto-grow dense rows="3" hide-details v-model="commentEditContent" color="blue-grey darken-2" :background-color="$vuetify.theme.dark ? `grey darken-5` : `white`"></v-textarea>
              <div class="d-flex align-center pt-3">
                <v-spacer></v-spacer>
                <v-btn class="mr-3" dark color="blue-grey darken-2" @click="editCommentCancel" outlined>
                  <v-icon left>mdi-close</v-icon><span class="text-none">{{$t('common:actions.cancel')}}</span>
                </v-btn>
                <v-btn dark color="blue-grey darken-2" @click="updateComment" depressed>
                  <v-icon left>mdi-comment</v-icon><span class="text-none">{{$t('common:comments.updateComment')}}</span>
                </v-btn>
              </div>
            </div>
          </v-card-text>
        </v-card>
      </v-timeline-item>
    </v-timeline>
    <div class="pt-5 text-center body-2 blue-grey--text" v-else-if="permissions.write">{{$t('common:comments.beFirst')}}</div>
    <div class="text-center body-2 blue-grey--text" v-else>{{$t('common:comments.none')}}</div>
    <v-dialog v-model="deleteCommentDialogShown" max-width="500">
      <v-card>
        <div class="dialog-header is-red">{{$t('common:comments.deleteConfirmTitle')}}</div>
        <v-card-text class="pt-5"><span>{{$t('common:comments.deleteWarn')}}</span>
          <div class="caption"><strong>{{$t('common:comments.deletePermanentWarn')}}</strong></div>
        </v-card-text>
        <v-card-chin>
          <v-spacer></v-spacer>
          <v-btn text @click="deleteCommentDialogShown = false">{{$t('common:actions.cancel')}}</v-btn>
          <v-btn color="red" dark @click="deleteComment">{{$t('common:actions.delete')}}</v-btn>
        </v-card-chin>
      </v-card>
    </v-dialog>
  </div>
</template>

<script>
import gql from 'graphql-tag'
import { get } from 'vuex-pathify'
import validate from 'validate.js'
import _ from 'lodash'

export default {
  data () {
    return {
      newcomment: '',
      isLoading: true,
      hasLoadedOnce: false,
      comments: [],
      guestName: '',
      guestEmail: '',
      commentToDelete: {},
      commentEditId: 0,
      commentEditContent: null,
      deleteCommentDialogShown: false,
      isBusy: false,
      scrollOpts: {
        duration: 1500,
        offset: 0,
        easing: 'easeInOutCubic'
      }
    }
  },
  computed: {
    pageId: get('page/id'),
    permissions: get('page/effectivePermissions@comments'),
    isAuthenticated: get('user/authenticated'),
    userDisplayName: get('user/name')
  },
  methods: {
    onIntersect (entries, observer, isIntersecting) {
      if (isIntersecting) {
        this.fetch(true)
      }
    },
    async fetch (silent = false) {
      this.isLoading = true
      try {
        const results = await this.$apollo.query({
          query: gql`
            query ($locale: String!, $path: String!) {
              comments {
                list(locale: $locale, path: $path) {
                  id
                  render
                  authorName
                  createdAt
                  updatedAt
                }
              }
            }
          `,
          variables: {
            locale: this.$store.get('page/locale'),
            path: this.$store.get('page/path')
          },
          fetchPolicy: 'network-only'
        })
        this.comments = _.get(results, 'data.comments.list', []).map(c => {
          const nameParts = c.authorName.toUpperCase().split(' ')
          let initials = _.head(nameParts).charAt(0)
          if (nameParts.length > 1) {
            initials += _.last(nameParts).charAt(0)
          }
          c.initials = initials
          return c
        })
      } catch (err) {
        console.warn(err)
        if (!silent) {
          this.$store.commit('showNotification', {
            style: 'red',
            message: err.message,
            icon: 'alert'
          })
        }
      }
      this.isLoading = false
      this.hasLoadedOnce = true
    },
    /**
     * Post New Comment
     */
    async postComment () {
      let rules = {
        comment: {
          presence: {
            allowEmpty: false
          },
          length: {
            minimum: 2
          }
        }
      }
      if (!this.isAuthenticated && this.permissions.write) {
        rules.name = {
          presence: {
            allowEmpty: false
          },
          length: {
            minimum: 2,
            maximum: 255
          }
        }
        rules.email = {
          presence: {
            allowEmpty: false
          },
          email: true
        }
      }
      const validationResults = validate({
        comment: this.newcomment,
        name: this.guestName,
        email: this.guestEmail
      }, rules, { format: 'flat' })

      if (validationResults) {
        this.$store.commit('showNotification', {
          style: 'red',
          message: validationResults[0],
          icon: 'alert'
        })
        return
      }

      try {
        const resp = await this.$apollo.mutate({
          mutation: gql`
            mutation (
              $pageId: Int!
              $replyTo: Int
              $content: String!
              $guestName: String
              $guestEmail: String
            ) {
              comments {
                create (
                  pageId: $pageId
                  replyTo: $replyTo
                  content: $content
                  guestName: $guestName
                  guestEmail: $guestEmail
                ) {
                  responseResult {
                    succeeded
                    errorCode
                    slug
                    message
                  }
                  id
                }
              }
            }
          `,
          variables: {
            pageId: this.pageId,
            replyTo: 0,
            content: this.newcomment,
            guestName: this.guestName,
            guestEmail: this.guestEmail
          }
        })

        if (_.get(resp, 'data.comments.create.responseResult.succeeded', false)) {
          this.$store.commit('showNotification', {
            style: 'success',
            message: this.$t('common:comments.postSuccess'),
            icon: 'check'
          })

          this.newcomment = ''
          await this.fetch()
          this.$nextTick(() => {
            this.$vuetify.goTo(`#comment-post-id-${_.get(resp, 'data.comments.create.id', 0)}`, this.scrollOpts)
          })
        } else {
          throw new Error(_.get(resp, 'data.comments.create.responseResult.message', 'An unexpected error occurred.'))
        }
      } catch (err) {
        this.$store.commit('showNotification', {
          style: 'red',
          message: err.message,
          icon: 'alert'
        })
      }
    },
    /**
     * Show Comment Editing Form
     */
    async editComment (cm) {
      this.$store.commit(`loadingStart`, 'comments-edit')
      this.isBusy = true
      try {
        const results = await this.$apollo.query({
          query: gql`
            query ($id: Int!) {
              comments {
                single(id: $id) {
                  content
                }
              }
            }
          `,
          variables: {
            id: cm.id
          },
          fetchPolicy: 'network-only'
        })
        this.commentEditContent = _.get(results, 'data.comments.single.content', null)
        if (this.commentEditContent === null) {
          throw new Error('Failed to load comment content.')
        }
      } catch (err) {
        console.warn(err)
        this.$store.commit('showNotification', {
          style: 'red',
          message: err.message,
          icon: 'alert'
        })
      }
      this.commentEditId = cm.id
      this.isBusy = false
      this.$store.commit(`loadingStop`, 'comments-edit')
    },
    /**
     * Cancel Comment Edit
     */
    editCommentCancel () {
      this.commentEditId = 0
      this.commentEditContent = null
    },
    /**
     * Update Comment with new content
     */
    async updateComment () {
      this.$store.commit(`loadingStart`, 'comments-edit')
      this.isBusy = true
      try {
        if (this.commentEditContent.length < 2) {
          throw new Error(this.$t('common:comments.contentMissingError'))
        }
        const resp = await this.$apollo.mutate({
          mutation: gql`
            mutation (
              $id: Int!
              $content: String!
            ) {
              comments {
                update (
                  id: $id,
                  content: $content
                ) {
                  responseResult {
                    succeeded
                    errorCode
                    slug
                    message
                  }
                  render
                }
              }
            }
          `,
          variables: {
            id: this.commentEditId,
            content: this.commentEditContent
          }
        })

        if (_.get(resp, 'data.comments.update.responseResult.succeeded', false)) {
          this.$store.commit('showNotification', {
            style: 'success',
            message: this.$t('common:comments.updateSuccess'),
            icon: 'check'
          })

          const cm = _.find(this.comments, ['id', this.commentEditId])
          cm.render = _.get(resp, 'data.comments.update.render', '-- Failed to load updated comment --')
          cm.updatedAt = (new Date()).toISOString()

          this.editCommentCancel()
        } else {
          throw new Error(_.get(resp, 'data.comments.delete.responseResult.message', 'An unexpected error occurred.'))
        }
      } catch (err) {
        console.warn(err)
        this.$store.commit('showNotification', {
          style: 'red',
          message: err.message,
          icon: 'alert'
        })
      }
      this.isBusy = false
      this.$store.commit(`loadingStop`, 'comments-edit')
    },
    /**
     * Show Delete Comment Confirmation Dialog
     */
    deleteCommentConfirm (cm) {
      this.commentToDelete = cm
      this.deleteCommentDialogShown = true
    },
    /**
     * Delete Comment
     */
    async deleteComment () {
      this.$store.commit(`loadingStart`, 'comments-delete')
      this.isBusy = true
      this.deleteCommentDialogShown = false

      try {
        const resp = await this.$apollo.mutate({
          mutation: gql`
            mutation (
              $id: Int!
            ) {
              comments {
                delete (
                  id: $id
                ) {
                  responseResult {
                    succeeded
                    errorCode
                    slug
                    message
                  }
                }
              }
            }
          `,
          variables: {
            id: this.commentToDelete.id
          }
        })

        if (_.get(resp, 'data.comments.delete.responseResult.succeeded', false)) {
          this.$store.commit('showNotification', {
            style: 'success',
            message: this.$t('common:comments.deleteSuccess'),
            icon: 'check'
          })

          this.comments = _.reject(this.comments, ['id', this.commentToDelete.id])
        } else {
          throw new Error(_.get(resp, 'data.comments.delete.responseResult.message', 'An unexpected error occurred.'))
        }
      } catch (err) {
        this.$store.commit('showNotification', {
          style: 'red',
          message: err.message,
          icon: 'alert'
        })
      }
      this.isBusy = false
      this.$store.commit(`loadingStop`, 'comments-delete')
    }
  }
}
</script>

<style lang="scss">
.comments-post {
  position: relative;

  &:hover {
    .comments-post-actions {
      opacity: 1;
    }
  }

  &-actions {
    position: absolute;
    top: 16px;
    right: 16px;
    opacity: 0;
    transition: opacity .4s ease;
  }

  &-content {
    > p:first-child {
      padding-top: 0;
    }

    p {
      padding-top: 1rem;
      margin-bottom: 0;
    }

    img {
      max-width: 100%;
      border-radius: 5px;
    }

    code {
      background-color: rgba(mc('pink', '500'), .1);
      box-shadow: none;
    }

    pre > code {
      margin-top: 1rem;
      padding: 12px;
      background-color: #111;
      box-shadow: none;
      border-radius: 5px;
      width: 100%;
      color: #FFF;
      font-weight: 400;
      font-size: .85rem;
      font-family: Roboto Mono, monospace;
    }
  }
}
</style>
