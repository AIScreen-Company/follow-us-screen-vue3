<template>
  <div
    id="screen"
    v-if="
      receivedMessage && receivedMessage.links && receivedMessage.links.length
    "
  >
    <div
      class="main-wrapper"
      :class="{
        bricks: receivedMessage.template === 'Bricks',
        default: receivedMessage.template !== 'Bricks',
      }"
    >
      <div
        class="background"
        :style="{
          backgroundImage:
            receivedMessage.currentBackground.link &&
            receivedMessage.template !== 'Bricks'
              ? `url(${receivedMessage.currentBackground.link})`
              : socialBg(activeLink)
              ? `url(${socialBg(activeLink)})`
              : 'none',
          opacity: receivedMessage.opacity ? receivedMessage.opacity / 100 : 1,
        }"
      ></div>
      <div class="header" v-if="receivedMessage.template !== 'Bricks'">
        <div
          class="website"
          v-if="
            receivedMessage.website &&
            receivedMessage.fieldsVisibility.value.includes('Website')
          "
        >
          <img :src="require('../assets/internet.svg')" alt="website" />
          {{ receivedMessage.website }}
        </div>
        <div class="contacts-wrapper">
          <div
            class="email"
            v-if="
              receivedMessage.email &&
              receivedMessage.fieldsVisibility.value.includes('Email')
            "
          >
            <img :src="require('../assets/email.svg')" alt="" />{{
              receivedMessage.email
            }}
          </div>
          <div
            class="phone"
            v-if="
              receivedMessage.phone &&
              receivedMessage.fieldsVisibility.value.includes('Phone')
            "
          >
            <img :src="require('../assets/phone.svg')" alt="" />{{
              receivedMessage.phone
            }}
          </div>
        </div>
      </div>
      <div class="header bricks" v-if="receivedMessage.template === 'Bricks'">
        <div class="title" v-if="receivedMessage.title">
          {{ receivedMessage.title }}
        </div>
        <div class="contacts-wrapper">
          <div
            class="email"
            v-if="
              receivedMessage.email &&
              receivedMessage.fieldsVisibility.value.includes('Email')
            "
          >
            <img :src="require('../assets/email.svg')" alt="" />{{
              receivedMessage.email
            }}
          </div>
          <div
            class="website"
            v-if="
              receivedMessage.website &&
              receivedMessage.fieldsVisibility.value.includes('Website')
            "
          >
            <img :src="require('../assets/internet.svg')" alt="website" />
            {{ receivedMessage.website }}
          </div>
          <div
            class="phone"
            v-if="
              receivedMessage.phone &&
              receivedMessage.fieldsVisibility.value.includes('Phone')
            "
          >
            <img :src="require('../assets/phone.svg')" alt="" />{{
              receivedMessage.phone
            }}
          </div>
        </div>
      </div>
      <div class="body default" v-if="receivedMessage.template === 'Default'">
        <div class="item">
          <div class="follow-us">
            <div class="text">
              {{ receivedMessage.title }}
            </div>
          </div>
          <div class="main-link">
            <img :src="socialLink(activeLink)" alt="" />{{
              accountName(activeLink)
            }}
          </div>
          <div class="qr-code">
            <qrcode-vue
              :value="activeLink"
              :size="150"
              level="H"
              render-as="svg"
              class="qrcode"
            ></qrcode-vue>
          </div>
        </div>
        <div class="running-line">
          <div class="slider" :style="sliderStyle"></div>
          <template v-for="(link, index) in links" :key="index">
            <div
              class="item"
              :style="{ width: 100 / links.length + '%' }"
              :class="{
                small:
                  links.length > 4 && receivedMessage.hideQrCodes === false,
                withoutQrCode: receivedMessage.hideQrCodes === true,
              }"
            >
              <div
                class="name-wrapper"
                :style="{
                  maxWidth:
                    links.length > 4 && receivedMessage.hideQrCodes === false
                      ? '60%'
                      : '80%',
                }"
              >
                <img :src="socialLink(link)" alt="" />
                <div class="link">{{ accountName(link) }}</div>
              </div>
              <div class="qr-code" v-if="receivedMessage.hideQrCodes !== true">
                <qrcode-vue
                  :value="link"
                  :size="100"
                  level="H"
                  render-as="svg"
                  class="qrcode"
                ></qrcode-vue>
              </div>
            </div>
          </template>
        </div>
      </div>
      <div class="body bricks" v-if="receivedMessage.template === 'Bricks'">
        <div
          class="item"
          v-for="(link, index) in links"
          :key="index"
          :class="{ active: activeLinkIndex === index }"
          :style="{
            width: getBricksItemWidth(index),
          }"
        >
          <div class="qr-code">
            <qrcode-vue
              :value="link"
              :size="150"
              level="H"
              render-as="svg"
              class="qrcode"
            ></qrcode-vue>
          </div>
          <div class="main-link">
            {{ accountName(link) }}
          </div>
          <img :src="socialLink(link)" alt="" class="icon" />
        </div>
      </div>
    </div>
  </div>

  <div class="loading" v-else>
    <img src="@/assets/loading.svg" alt="loading" />
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount, computed } from 'vue'
import qrcodeVue from 'qrcode.vue'

const receivedMessage = ref(null)
const activeLinkIndex = ref(0)
let intervalId = null

const sliderStyle = computed(() => {
  const linksArray = links.value // Получаем значение вычисляемого свойства
  const itemWidth = 100 / linksArray.length
  return {
    width: `${itemWidth}%`,
    transform: `translateX(${activeLinkIndex.value * 100}%)`,
  }
})
const links = computed(() => {
  if (!receivedMessage.value || !receivedMessage.value.links) return []
  return receivedMessage.value.links.filter((link) => link !== null)
})

const activeLink = computed(() => {
  if (!receivedMessage.value || !receivedMessage.value.links) return ''
  return links.value[activeLinkIndex.value]
})

const socialLink = (url) => {
  try {
    if (!url) throw new Error('Invalid URL')

    url = url.trim()
    if (!url.startsWith('http://') && !url.startsWith('https://')) {
      url = `https://${url}`
    }

    const parsedUrl = new URL(url)
    const hostname = parsedUrl.hostname.toLowerCase()
    const pathname = parsedUrl.pathname.toLowerCase()

    const paths = {
      youtube: /^(?:www\.)?youtube\.com/i,
      instagram: /^(?:www\.)?instagram\.com/i,
      facebook: /^(?:www\.)?facebook\.com/i,
      linkedin: /^(?:www\.)?linkedin\.com/i,
      twitter: /^(?:www\.)?twitter\.com/i,
      x: /^(?:www\.)?x\.com/i,
      vimeo: /^(?:www\.)?vimeo\.com/i,
      pinterest: /^(?:www\.)?pinterest\.com/i,
      flickr: /^(?:www\.)?flickr\.com/i,
      snapchat: /^(?:www\.)?snapchat\.com/i,
      github: /^(?:www\.)?github\.com/i,
      medium: /^(?:www\.)?medium\.com/i,
      dribbble: /^(?:www\.)?dribbble\.com/i,
    }

    let icon = 'internet.svg'
    for (const platform in paths) {
      const isLinkedin = platform === 'linkedin'

      if (
        (isLinkedin &&
          paths[platform].test(hostname) &&
          /^(\/in|\/company|\/pub)\//i.test(pathname)) ||
        (!isLinkedin && paths[platform].test(hostname))
      ) {
        icon = `${platform}.svg`
        break
      }
    }

    return require(`@/assets/${icon}`)
  } catch (error) {
    return require(`@/assets/internet.svg`)
  }
}

const accountName = (url) => {
  const patterns = {
    facebook: /facebook\.com\/(?:profile\.php\?id=)?([^\/?]+)/,
    instagram: /instagram\.com\/([^\/?]+)/,
    x: /x\.com\/([^\/?]+)/,
    youtube: /youtube\.com\/(?:channel\/|user\/)?([^\/?]+)/,
    skype: /skype\.com\/([^\/?]+)/,
    linkedin: /linkedin\.com\/(?:in|company)\/([^\/?]+)/,
    vimeo: /vimeo\.com\/([^\/?]+)/,
    pinterest: /pinterest\.com\/([^\/?]+)/,
    flickr: /flickr\.com\/photos\/([^\/?]+)/,
    snapchat: /snapchat\.com\/add\/([^\/?]+)/,
    github: /github\.com\/([^\/?]+)/,
    medium: /medium\.com\/@?([^\/?]+)/,
    dribbble: /dribbble\.com\/([^\/?]+)/,
  }

  for (const platform in patterns) {
    const match = url.match(patterns[platform])
    if (match && match[1]) {
      return match[1].includes('@') ? match[1] : `@${match[1]}`
    }
  }
  return url
}

const socialBg = (url) => {
  try {
    if (!url) throw new Error('Invalid URL')

    url = url.trim()
    if (!url.startsWith('http://') && !url.startsWith('https://')) {
      url = `https://${url}`
    }

    const parsedUrl = new URL(url)
    const hostname = parsedUrl.hostname.toLowerCase()
    const pathname = parsedUrl.pathname.toLowerCase()

    const paths = {
      youtube: /^(?:www\.)?youtube\.com/i,
      instagram: /^(?:www\.)?instagram\.com/i,
      facebook: /^(?:www\.)?facebook\.com/i,
      linkedin: /^(?:www\.)?linkedin\.com/i,
      twitter: /^(?:www\.)?twitter\.com/i,
      x: /^(?:www\.)?x\.com/i,
      vimeo: /^(?:www\.)?vimeo\.com/i,
      pinterest: /^(?:www\.)?pinterest\.com/i,
      flickr: /^(?:www\.)?flickr\.com/i,
      snapchat: /^(?:www\.)?snapchat\.com/i,
      github: /^(?:www\.)?github\.com/i,
      medium: /^(?:www\.)?medium\.com/i,
      dribbble: /^(?:www\.)?dribbble\.com/i,
    }

    let icon = ''
    for (const platform in paths) {
      const isLinkedin = platform === 'linkedin'

      if (
        (isLinkedin &&
          paths[platform].test(hostname) &&
          /^(\/in|\/company|\/pub)\//i.test(pathname)) ||
        (!isLinkedin && paths[platform].test(hostname))
      ) {
        icon = `${platform}-bg.png`
        break
      }
    }

    return require(`@/assets/${icon}`)
  } catch (error) {
    return
  }
}

const receiveMessage = (event) => {
  if (
    !event.data ||
    !event.data.appSettings ||
    !event.data.appSettings.settings
  )
    return
  stopLinkRotation()
  receivedMessage.value = JSON.parse(event.data.appSettings.settings)
  startLinkRotation()
}

const startLinkRotation = () => {
  if (
    !receivedMessage.value ||
    !receivedMessage.value.timer ||
    !links.value.length
  ) {
    setTimeout(() => {
      startLinkRotation()
    }, 1000)
    return
  }

  intervalId = setInterval(() => {
    activeLinkIndex.value = (activeLinkIndex.value + 1) % links.value.length
  }, receivedMessage.value.timer * 1000)
}

const stopLinkRotation = () => {
  clearInterval(intervalId)
}

const getBricksItemWidth = (index) => {
  const totalLinks = links.value.length
  const firstRowCount = links.value.length > 3 ? 3 : links.value.length
  const secondRowCount = totalLinks - firstRowCount

  if (index < firstRowCount) {
    return `${100 / firstRowCount - 1}%`
  } else {
    if (secondRowCount === 1) {
      return '100%'
    } else if (secondRowCount === 2) {
      return '49%'
    } else if (secondRowCount === 3) {
      return '32.33%'
    }
  }
}

onMounted(() => {
  window.addEventListener('message', receiveMessage)
  startLinkRotation()
})

onBeforeUnmount(() => {
  window.removeEventListener('message', receiveMessage)
  stopLinkRotation()
})
</script>

<style lang="scss" scoped>
@function vw($value) {
  @return $value / 1920 * 100vw;
}
.loading {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100%;
  img {
    width: 30%;
  }
}
#screen {
  width: 100%;
  height: 100%;
  background-color: #f5f5f8;
  padding: vw(40);
  overflow: hidden;
  .main-wrapper {
    box-shadow: 0px vw(8) vw(32) 0px #16151529;
    background: #ffffff;
    height: 100%;
    overflow: hidden;
    position: relative;
    z-index: 1;

    .background {
      background-size: cover;
      background-repeat: no-repeat;
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      z-index: -1;
    }
    &.default {
      border-radius: vw(30);
    }
    &.bricks {
      box-shadow: none;
      background: transparent !important;
      display: flex;
      flex-direction: column;
    }
    .header {
      display: flex;
      flex-direction: row;
      justify-content: space-between;
      align-items: center;
      margin-bottom: vw(45);
      padding: vw(33) vw(36) 0 vw(36);
      &.bricks {
        background: linear-gradient(
          272.02deg,
          #6640f5 -2.92%,
          #8768f6 30.53%,
          #0071e2 61.08%,
          #6cb7f6 99.41%
        ) !important;
        border-radius: vw(17);
        padding: vw(8) vw(18);
        margin-bottom: vw(31);
        .title {
          font-size: vw(40);
          line-height: vw(48);
          letter-spacing: -0.02em;
          font-weight: 700;
          color: #f5f5f8;
        }
        .website,
        .contacts-wrapper {
          color: #f5f5f8;
          font-weight: 700 !important;
          width: unset !important;
          img {
            filter: invert(1);
          }
        }
      }
      .website {
        display: flex;
        flex-direction: row;
        gap: vw(10);
        font-weight: 700;
        font-size: vw(24);
        letter-spacing: -0.02em;
        color: #14121f;
        display: flex;
        flex-direction: row;
        align-items: center;
        gap: vw(8);
        img {
          width: vw(20);
          height: vw(20);
        }
      }
      .contacts-wrapper {
        display: flex;
        flex-direction: row;
        width: 100%;
        justify-content: flex-end;
        gap: vw(40);
        align-items: center;
        font-size: vw(24);
        letter-spacing: -0.02em;
        color: #14121f;
        .email,
        .phone {
          display: flex;
          flex-direction: row;
          align-items: center;
          font-weight: 700;
          gap: vw(8);
          img {
            width: vw(24);
            height: vw(24);
          }
        }
      }
    }
    .body.default {
      display: flex;
      flex-direction: column;
      align-items: center;
      flex-wrap: wrap;
      height: 100%;
      width: 100%;
      overflow: hidden;
      transition: all 0.3s ease;
      height: calc(100vh - vw(150));
      .item {
        min-width: 100% !important;
        height: 80% !important;
        display: flex;
        flex-direction: column;
        align-items: center;
        transition: all 0.3s ease;

        .follow-us {
          background: #f5f5f8;
          border-radius: vw(72);

          padding: vw(10) vw(40);
          margin-bottom: vw(55);
          .text {
            font-weight: 700;
            font-size: vw(64);
            line-height: vw(72);
            letter-spacing: -0.02em;
            background: linear-gradient(
              272.02deg,
              #6640f5 -2.92%,
              #8768f6 30.53%,
              #0071e2 61.08%,
              #6cb7f6 99.41%
            );
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
          }
        }
        .main-link {
          display: flex;
          flex-direction: row;
          justify-content: center;
          align-items: center;
          font-weight: 700;
          font-size: vw(64);
          line-height: vw(72);
          letter-spacing: -0.02em;
          gap: vw(31);
          margin-bottom: vw(31);
          img {
            width: vw(90);
            height: vw(90);
          }
        }
        .qr-code {
          height: vw(320);
          width: vw(320);
          .qrcode {
            height: vw(320);
            width: vw(320);
            ::v-deep {
              svg {
                height: vw(320) !important;
                width: vw(320) !important;
              }
            }
          }
        }
      }
      .running-line {
        display: flex;
        flex-direction: row;
        height: 20%;
        width: 100%;
        flex-wrap: nowrap;
        padding: vw(0) vw(13) vw(25) vw(13);
        gap: vw(31);
        position: relative;
        .slider {
          position: absolute;
          top: 0;
          left: 0;
          height: 70%;
          background: #f5f5f8;
          transition: transform 0.3s ease;
          border-radius: vw(17);
        }
        .item {
          display: flex;
          flex-direction: row;
          align-items: center;
          overflow: hidden;
          width: 100%;
          min-width: unset !important;
          padding: vw(13);
          border-radius: vw(17);
          justify-content: space-between;
          transition: all 0.3s ease;
          position: relative;
          &.active {
            background: #f5f5f8;
            transition: all 0.3s ease;
          }

          .name-wrapper {
            display: flex;
            flex-direction: row;
            align-items: center;
            gap: vw(10);
            max-width: 60%;
            padding-right: vw(10);
            .link {
              max-width: 100%;

              overflow: hidden;
              text-overflow: ellipsis;
              white-space: nowrap;
              font-size: vw(24);
              line-height: vw(29);
              font-weight: 700;
              letter-spacing: -0.02em;
              color: #14121f;
            }
            img {
              width: vw(80);
              height: vw(80);
            }
          }
          .qr-code {
            height: vw(100);
            width: vw(100);
            .qrcode {
              height: vw(100);
              width: vw(100);
              ::v-deep {
                svg {
                  height: vw(100) !important;
                  width: vw(100) !important;
                }
              }
            }
          }
          &.withoutQrCode {
            .name-wrapper {
              max-width: 95% !important;
            }
          }
          &.small {
            .name-wrapper {
              align-items: flex-start !important;
              flex-direction: column !important;
            }
            .link {
            }
            img {
              width: vw(40) !important;
              height: vw(40) !important;
            }
          }
        }
      }
    }
    .body.bricks {
      display: flex;
      flex-direction: row;
      align-items: center;
      flex-wrap: wrap;
      justify-content: space-between;
      width: 100%;
      overflow: hidden;
      transition: all 0.3s ease;
      background: none !important;
      gap: vw(24);
      flex-grow: 1;
      align-items: stretch;
      .item {
        min-width: 31.3% !important;
        display: flex;
        flex-direction: column;
        align-items: center;
        position: relative;
        transition: all 0.3s ease;
        justify-content: center;
        overflow: hidden;
        background: #fff;
        border-radius: vw(24);
        .main-link {
          font-weight: 700;
          font-size: vw(40);
          line-height: vw(48);
          letter-spacing: -0.02em;
          overflow: hidden;
          text-overflow: ellipsis;
          max-width: 65%;
          img {
            width: vw(90);
            height: vw(90);
          }
        }
        .qr-code {
          transition: all 0.3s ease;
          margin-bottom: vw(31);
          .qrcode {
            height: vw(200);
            width: vw(200);
            transition: all 0.3s ease;

            ::v-deep {
              svg {
                height: vw(200) !important;
                transition: all 0.3s ease;

                width: vw(200) !important;
              }
            }
          }
        }
        .icon {
          position: absolute;
          width: vw(64);
          height: vw(64);
          bottom: vw(24);
          left: vw(24);
          z-index: 1;
        }
        &.active {
          .qr-code {
            margin-bottom: vw(17);
            .qrcode {
              height: vw(280);
              width: vw(280);
              transition: all 0.3s ease;

              ::v-deep {
                svg {
                  height: vw(280) !important;
                  transition: all 0.3s ease;

                  width: vw(280) !important;
                }
              }
            }
          }
        }
      }
    }
  }
}
</style>
