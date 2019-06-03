<!-- This is a Vue.js single file component. -->
<!-- Check the Vue.js doc here :  -->
<!-- https://vuejs.org/v2/guide/ -->

<!-- This is your HTML -->
<template>
    <div>
        <!-- wwManager:start -->
        <wwSectionEditMenu :sectionCtrl="sectionCtrl" :options="openOptions"></wwSectionEditMenu>
        <!-- wwManager:end -->
        <wwObject class="background" :ww-object="section.data.bg" ww-category="background"></wwObject>
        <!--TOP WWOBJS-->
        <div class="top-ww-objs">
            <wwLayoutColumn
                tag="div"
                ww-default="ww-image"
                :ww-list="section.data.topWwObjs"
                class="top-ww-obj"
                @ww-add="add(section.data.topWwObjs, $event)"
                @ww-remove="remove(section.data.topWwObjs, $event)"
            >
                <wwObject v-for="topWwObj in section.data.topWwObjs" :key="topWwObj.uniqueId" :ww-object="topWwObj"></wwObject>
            </wwLayoutColumn>
        </div>

        <div class="container hidden-mobile">
            <div class="container-center">
                <div class="thumbnail-container" v-for="(feature, index) in section.data.features" :key="feature.uniqueId" :style="columnWidth">
                    <!-- wwManager:start -->
                    <wwContextMenu
                        tag="div"
                        class="contextmenu"
                        v-if="editMode"
                        @ww-add-before="addFeature(index, 'before')"
                        @ww-add-after="addFeature(index, 'after')"
                        @ww-remove="removeFeature(index)"
                    >
                        <div class="wwi wwi-config"></div>
                    </wwContextMenu>
                    <!-- wwManager:end -->
                    <wwObject class="background" :ww-object="feature.background" ww-category="background"></wwObject>

                    <div class="forehead-banner">
                        <wwObject tag="div" class="forehead" :ww-object="feature.banner" ww-default="ww-image"></wwObject>
                    </div>
                    <!-- team row -->
                    <div class="team-pic-container" :ww-list="feature.teamRow">
                        <wwObject
                            tag="div"
                            class="team-pic"
                            v-for="(teamPic, index) in feature.teamRow"
                            :key="index"
                            :ww-object="teamPic"
                            @ww-add-before="addElement(feature.teamRow, index, 'before')"
                            @ww-add-after="addElement(feature.teamRow, index, 'after')"
                            @ww-remove="removeElement(feature.teamRow, index)"
                        ></wwObject>
                    </div>

                    <wwLayoutColumn tag="div" ww-default="ww-image" :ww-list="feature.contents" class="content" @ww-add="add(feature.contents, $event)" @ww-remove="remove(feature.contents, $event)">
                        <wwObject tag="div" v-for="content in feature.contents" :key="content.uniqueId" :ww-object="content"></wwObject>
                    </wwLayoutColumn>
                </div>
            </div>
        </div>

        <v-touch
            ref="swiper"
            :enabled="!editMode"
            @swipeleft="nextSlide()"
            @swiperight="prevSlide()"
            :swipe-options="{ direction: 'horizontal', threshold: 10, velocity: 0.2 }"
            class="container mobile-wrapper"
        >
            <div class="container-center" :style="[mobileStyle, mobileTransition]">
                <div class="thumbnail-container" v-for="feature in section.data.features" :key="feature.uniqueId" :style="cardWidth">
                    <!-- wwManager:start -->
                    <wwContextMenu
                        tag="div"
                        class="contextmenu contextmenu-center"
                        v-if="editMode"
                        @ww-add-before="addFeature(index, 'before')"
                        @ww-add-after="addFeature(index, 'after')"
                        @ww-remove="removeFeature(index)"
                    >
                        <div class="wwi wwi-config"></div>
                    </wwContextMenu>
                    <!-- wwManager:end -->
                    <wwObject class="background" :ww-object="feature.background" ww-category="background"></wwObject>

                    <div class="forehead-banner">
                        <wwObject tag="div" class="forehead" :ww-object="feature.banner" ww-default="ww-image"></wwObject>
                    </div>
                    <!-- team row -->
                    <div class="team-pic-container" :ww-list="feature.teamRow">
                        <wwObject
                            tag="div"
                            class="team-pic"
                            v-for="(teamPic, index) in feature.teamRow"
                            :key="index"
                            :ww-object="teamPic"
                            @ww-add-before="addElement(feature.teamRow, index, 'before')"
                            @ww-add-after="addElement(feature.teamRow, index, 'after')"
                            @ww-remove="removeElement(feature.teamRow, index)"
                        ></wwObject>
                    </div>

                    <wwLayoutColumn tag="div" ww-default="ww-image" :ww-list="feature.contents" class="content" @ww-add="add(feature.contents, $event)" @ww-remove="remove(feature.contents, $event)">
                        <wwObject tag="div" v-for="content in feature.contents" :key="content.uniqueId" :ww-object="content"></wwObject>
                    </wwLayoutColumn>
                </div>
            </div>
            <div class="content-dots-wrapper">
                <li
                    v-for="(dot, index) in section.data.features"
                    class="content-dot"
                    :style="{'background': ((sliderPosition == index) ? section.data.dotColor : ''), 'border-color': section.data.dotColor}"
                    :key="dot.uniqueId"
                >
                    <div class="dot" @click="switchToIndex(sliderPosition, index)"></div>
                </li>
            </div>
        </v-touch>

        <!--BOTTOM WWOBJS-->
        <div class="bottom-ww-objs">
            <wwLayoutColumn
                tag="div"
                ww-default="ww-image"
                :ww-list="section.data.bottomWwObjs"
                class="top-ww-obj"
                @ww-add="add(section.data.bottomWwObjs, $event)"
                @ww-remove="remove(section.data.bottomWwObjs, $event)"
            >
                <wwObject v-for="bottomWwObj in section.data.bottomWwObjs" :key="bottomWwObj.uniqueId" :ww-object="bottomWwObj"></wwObject>
            </wwLayoutColumn>
        </div>
    </div>
</template>

<!-- This is your Javascript -->
<!-- ✨ Here comes the magic ✨ -->
<script>

const VueTouch = require('vue-touch')

Vue.use(VueTouch, { name: 'v-touch' })
import { getNewFeature } from "./defaultFeature"

export default {
    name: "__COMPONENT_NAME__",
    props: {
        // The section controller object is passed to you.
        sectionCtrl: Object
    },
    data() {
        return {
            maxThumbnailsPerLine: 4,
            columnWidth: { 'width': 'calc(33.33% - 30px)' },
            sliderPosition: 0,


        }
    },
    computed: {
        //Get the section object here !
        // It contains all the info and data about the section
        // Use it has you like !
        section() {
            return this.sectionCtrl.get();
        },
        editMode() {
            return this.sectionCtrl.getEditMode() == 'CONTENT'
        },

        featuresLength() {
            return this.section.data.features.length
        },
        mobileStyle() {
            return { 'width': 'calc(' + this.featuresLength + ' * 100%)' }
        },
        mobileTransition() {
            return { 'transform': 'translate(calc(' + (- this.sliderPosition * (100 / this.featuresLength)) + '% + ' + this.sliderPosition * 32 + 'px ), 0)' }
        },
        cardWidth() {
            return { 'width': 'calc(' + (100 / this.featuresLength) + '% - 50px)' }
        }
    },

    created() {

        let needUpdate = false
        //Initialize section data
        this.section.data = this.section.data || {};
        this.section.data.dotColor = this.section.data.dotColor || "#9013FE"

        if (!this.section.data.bg) {
            this.section.data.bg = wwLib.wwObject.getDefault({
                type: 'ww-color'
            });
            needUpdate = true
        }

        if (!this.section.data.features) {
            this.section.data.features = []
            needUpdate = true
        }
        if (!this.section.data.thumbnailsPerLine) {
            this.section.data.thumbnailsPerLine = 4;
            needUpdate = true;
        }

        if (_.isEmpty(this.section.data.features)) {
            for (let i = 0; i < 3; i++) {
                this.section.data.features.push(
                    {
                        background: wwLib.wwObject.getDefault({ type: 'ww-color', data: { color: 'white' } }),
                        banner: wwLib.wwObject.getDefault({ type: 'ww-image' }),
                        contents: [],
                        teamRow: [
                            wwLib.wwObject.getDefault({
                                type: "ww-image",
                            })
                        ]
                    },
                )
            }

            needUpdate = true;
        }

        if (_.isEmpty(this.section.data.topWwObjs)) {
            this.section.data.topWwObjs = [];
            needUpdate = true;
        }
        if (_.isEmpty(this.section.data.bottomWwObjs)) {
            this.section.data.bottomWwObjs = [];
            needUpdate = true;
        }

        if (needUpdate) {
            this.sectionCtrl.update(this.section);
        }
    },
    mounted() {
        this.init();
    },
    beforeDestroy() {
        window.removeEventListener("resize", this.setThumbnailsPerLine);
    },
    methods: {
        init() {
            this.setThumbnailsPerLine();
            window.addEventListener("resize", this.setThumbnailsPerLine);
        },
        nextSlide() {
            try {
                this.sliderPosition++
                if (this.sliderPosition > this.featuresLength - 1)
                    this.sliderPosition = 0
                this.$forceUpdate()
            } catch (error) {
                wwLib.wwLog.error('ERROR : ', error);
            }
        },

        prevSlide() {
            try {
                this.sliderPosition--
                if (this.sliderPosition < 0)
                    this.sliderPosition = this.featuresLength - 1
                this.$forceUpdate()
            } catch (error) {
                wwLib.wwLog.error('ERROR : ', error);
            }

        },

        switchToIndex(index, position) {
            try {
                if (position < this.section.data.features.length && index != position) {
                    this.sliderPosition = position
                }
            } catch (error) {
                wwLib.wwLog.error('ERROR : ', error);

            }

        },

        setThumbnailsPerLine() {
            try {
                let width = window.innerWidth;
                if (width < 576) {
                    this.maxThumbnailsPerLine = 1;
                }
                else if (width < 992) {
                    this.maxThumbnailsPerLine = 2;
                }
                else if (width < 1200) {
                    this.maxThumbnailsPerLine = 3;
                }
                else {
                    this.maxThumbnailsPerLine = 4;
                }

                switch (Math.min(this.section.data.thumbnailsPerLine, this.maxThumbnailsPerLine)) {
                    case 1:
                        this.columnWidth = { 'width': "calc(100% - 30px)" };
                        break;
                    case 2:
                        this.columnWidth = { 'width': "calc(50% - 30px)" };
                        break;
                    case 3:
                        this.columnWidth = { 'width': "calc(33.3333% - 30px)" };
                        break;
                    default:
                        this.columnWidth = { 'width': "calc(25% - 30px)" };
                        break;
                }
            } catch (error) {
                wwLib.wwLog.error('ERROR : ', error);
            }
        },
        /* wwManager:start */
        add(list, options) {
            try {
                list.splice(options.index, 0, options.wwObject);
                this.sectionCtrl.update(this.section);
            } catch (error) {
                wwLib.wwLog.error('ERROR : ', error);
            }
        },


        /* add a new section to the slider */
        addFeature(_index, where) {
            try {
                const up = (where == 'after') ? parseInt(1) : 0;
                const index = _index + up
                const newCard = getNewFeature()

                this.section.data.features.splice(index, 0, newCard);
                this.sectionCtrl.update(this.section);
            } catch (err) {
                wwLib.wwLog.error('ERROR : ', error);
            }
        },
        removeFeature(index) {
            try {
                this.section.data.features.splice(index, 1);
                if (!this.section.data.features.length) {
                    this.addFeature(0, 'after');
                }
                this.sectionCtrl.update(this.section);
                this.prevSlide()
            } catch (error) {
                wwLib.wwLog.error('ERROR : ', error);

            }
        },

        /* add picture */

        addElement(list, _index, where) {
            try {
                const up = (where == 'after') ? parseInt(1) : 0;
                const index = _index + up
                let newCopie = JSON.parse(JSON.stringify(list[0]))

                wwLib.wwUtils.changeUniqueIds(newCopie)
                list.splice(index, 0, newCopie);
                this.sectionCtrl.update(this.section);
            } catch (error) {
                wwLib.wwLog.error('ERROR : ', error);
            }
        },

        removeElement(list, index) {
            try {
                if (list.length > 1) {
                    list.splice(index, 1);
                    this.sectionCtrl.update(this.section);
                }
            } catch (error) {
                wwLib.wwLog.error('ERROR : ', error);
            }
        },

        async openOptions() {
            try {
                wwLib.wwPopups.addStory('WWSLIDER_CUSTOM', {
                    title: {
                        en: 'Fill in code',
                        fr: 'Inserer le code'
                    },
                    type: 'wwPopupForm',
                    storyData: {
                        fields: [
                            {
                                label: {
                                    en: 'Column per line',
                                    fr: 'Nombre de colonnes par ligne :'
                                },
                                type: 'text',
                                key: 'columnPerLine',
                                valueData: 'section.data.thumbnailsPerLine',
                                desc: {
                                    en: 'The number of column per line',
                                    fr: 'Le nombre de colonnes par ligne'
                                }

                            },
                            {
                                label: {
                                    en: 'Navigation dots color:',
                                    fr: 'Couleur des points de navigations :'
                                },
                                type: 'color',
                                key: 'dotsColor',
                                valueData: 'section.data.dotColor',
                                desc: {
                                    en: 'Choose a Nav dots color:',
                                    fr: 'Changer la couleur des points de navigations :'
                                },
                            },

                        ]
                    },
                    buttons: {
                        NEXT: {
                            text: {
                                en: 'Finish',
                                fr: 'Terminer'
                            },
                            next: null
                        }
                    }
                })
                let options = {
                    firstPage: 'WWSLIDER_CUSTOM',
                    data: {
                        columnPerLine: this.section.data.thumbnailsPerLine,
                        section: this.section,

                    },
                }

                const result = await wwLib.wwPopups.open(options)
                if (typeof (result) != 'undefined') {
                    if (typeof (result.dotsColor) != 'undefined') {
                        this.section.data.dotColor = result.dotsColor
                    }
                    if (result.columnPerLine) {
                        this.section.data.thumbnailsPerLine = result.columnPerLine;
                        this.sectionCtrl.update(this.section);
                        this.setThumbnailsPerLine();
                    }
                    this.sectionCtrl.update(this.section);
                }
            } catch (error) {
                wwLib.wwLog.error('ERROR : ', error);
            }
        },
        /* Used on mobile version to swipe */
        remove(list, options) {
            try {
                list.splice(options.index, 1);
                this.sectionCtrl.update(this.section);
            } catch (error) {
                wwLib.wwLog.error('ERROR : ', error);
            }

        },

        /* wwManager:end */



    }
};
</script>

<!-- This is your CSS -->
<!-- Add "scoped" attribute to limit CSS to this component only -->
<!-- Add lang="scss" or others to use computed CSS -->
<style lang='scss' scoped>
.background {
    position: absolute;
    top: 0;
    left: 0;
    height: 100%;
    width: 100%;
}

.top-ww-objs,
.bottom-ww-objs {
    position: relative;
    .top-ww-obj,
    .bottom-ww-obj {
        position: relative;
    }
}

.container {
    position: relative;
    @media (min-width: 768px) {
        width: 80%;
    }

    @media (min-width: 1200px) {
        width: 70%;
    }
    .container-center {
        display: flex;
        transition: transform 0.5s ease;
        @media (min-width: 1024px) {
            justify-content: center;
            flex-wrap: wrap;
        }
        .thumbnail-container {
            width: 30%;
            position: relative;
            margin: 30px 0 30px 15px;
            background-color: white;
            min-height: 50px;
            box-shadow: 0 10px 40px 0 rgba(113, 124, 137, 0.2);
            border-radius: 7px;
            overflow: hidden;
            transition: transform 0.4s ease-out, box-shadow 0.4s ease-out;
            .forehead-banner {
                position: relative;
            }
            .background {
                border-radius: 7px;
                overflow: hidden;
            }
            .team-pic-container {
                margin-top: -50px;
                display: flex;
                flex-direction: row;
                justify-content: flex-start;
                align-items: center;
                .team-pic {
                    width: 90px;
                    height: 90px;
                }
                .team-pic:first-child {
                    margin-left: 15px;
                }
                .team-pic:last-child {
                    margin-right: 15px;
                }
                .team-pic:not(:first-child) {
                    margin-left: -20px;
                }
            }

            .content {
                position: relative;
            }
            /* wwManager:start */

            .contextmenu {
                position: absolute;
                top: 0;
                left: 0;
                width: 30px;
                height: 30px;
                color: white;
                background-color: #ef811a;
                border-radius: 100%;
                display: flex;
                justify-content: center;
                align-items: center;
                font-size: 1.2rem;
                cursor: pointer;
                z-index: 2;
            }
            .contextmenu-center {
                left: calc(50% - 15px);
            }
            /* wwManager:end */
        }
        .thumbnail-container:first-child {
            margin-left: 25px;
        }
    }
}

.content-dots-wrapper {
    display: flex;
    list-style: none;
    justify-content: center;
    position: relative;
    padding-bottom: 15px;
    margin-top: 20px;

    .content-dot {
        margin-right: 15px;
        border-radius: 100%;
        border-width: 2px;
        border-style: solid;
        .dot {
            cursor: pointer;
            width: 15px;
            height: 15px;
            pointer-events: all;
        }
    }
}
.hidden-mobile {
    display: none;
    @media (min-width: 1024px) {
        display: block;
    }
}
.mobile-wrapper {
    display: block;
    position: relative;
    @media (min-width: 1024px) {
        display: none;
    }
}
</style>
