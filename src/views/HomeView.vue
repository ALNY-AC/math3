<template>
    <div class="home-view">
        <div class="container">
            <el-card header="数学题生成器">
                <!-- 数学题生成器 -->
                <el-form>
                    <el-form-item label="题目总数">
                        <el-input-number v-model="itemCount"></el-input-number>
                    </el-form-item>
                    <!-- 运算范围 -->
                    <el-form-item label="运算范围">
                        <el-input-number v-model="countRange"></el-input-number>
                    </el-form-item>
                    <!-- 题目类型复选 -->
                    <el-form-item label="题目类型">
                        <el-checkbox-group v-model="types">
                            <el-checkbox value="+">加法</el-checkbox>
                            <el-checkbox value="-">减法</el-checkbox>
                            <el-checkbox value="*">乘法</el-checkbox>
                            <el-checkbox value="÷">除法</el-checkbox>
                        </el-checkbox-group>
                    </el-form-item>
                    <!-- 生成按钮 -->
                    <el-form-item>
                        <el-button type="primary" @click="generate">生成</el-button>
                    </el-form-item>
                </el-form>


            </el-card>
            <el-card>
                <div v-if="exams.length > 0">

                    <!-- 题目列表 -->

                    <div class="exam-list">
                        <div class="exam-item" v-for="(item, i) in exams">
                            <div class="exam-label">
                                {{ item.question }}
                                <span v-if="typeof item.result === 'boolean'"> = {{ item.correctAnswer }}</span>
                            </div>
                            <div class="exam-content">
                                <el-input v-model="item.userAnswer">
                                    <template #append v-if="typeof item.result === 'boolean'">
                                        <!-- 对错的 -->
                                        <el-icon v-if="item.result" color="#67c23a">
                                            <Select />
                                        </el-icon>
                                        <el-icon v-else color="#f56c6c">
                                            <CloseBold />
                                        </el-icon>
                                    </template>

                                </el-input>
                            </div>
                        </div>
                    </div>
                    <div style="text-align: center;padding:20px 0">
                        <el-button type="primary" @click="submit">提交</el-button>
                    </div>

                </div>
                <el-empty v-else description="请生成题目"></el-empty>
            </el-card>
        </div>
    </div>
</template>
<script setup lang="ts">
import { ref, reactive } from 'vue'
import * as mathjs from 'mathjs'
import { CloseBold, Select } from '@element-plus/icons-vue';
const itemCount = ref(10)
const countRange = ref(10)
const types = ref(['+', '-', '*', '÷'])


class Exam {
    // 题目数字
    numbers: number[]
    // 用户答案
    userAnswer: string = ''
    // 正确答案
    get correctAnswer() {
        return this.calc()
    }
    // 题目类型
    type: string
    // 对错
    result: boolean | null = null
    // 题目 get
    get question() {
        return this.numbers.join(' ' + this.type + ' ')
    }
    constructor(numbers: number[], type: string) {
        this.numbers = numbers
        this.type = type
    }
    // 计算题目结果
    calc() {
        return mathjs.evaluate(this.numbers.join(this.type == '÷' ? '/' : this.type)).toFixed(2).replace(/\.00$/, '')
    }
    // 校验答案
    check() {
        // 如果没写则直接错误
        if (this.userAnswer.length <= 0) {
            this.result = false
            return
        }
        const result = this.calc()
        this.result = !!mathjs.equal(result, Number(this.userAnswer))
    }

}

const exams = ref<Exam[]>([])


// 获取随机数字，范围为 0～range
const getRandomNumber = (range: number) => {
    return Math.floor(Math.random() * range)
}


// 生成题目
const generate = () => {
    exams.value = []
    for (let i = 0; i < itemCount.value; i++) {
        let a = getRandomNumber(countRange.value)
        let b = getRandomNumber(countRange.value)
        const type = types.value[getRandomNumber(types.value.length)]
        // 防止除数为0
        if (type == '÷' && b == 0) {
            b = Math.max(1, getRandomNumber(countRange.value))
        }
        const numbers = [a, b]
        exams.value.push(new Exam(numbers, type))
    }
}

const submit = () => {
    exams.value.forEach(item => {
        item.check()
    })
}

</script>
<style scoped lang="scss">
.home-view {
    padding: 10px;
}

.exam-list {
    display: grid;
    // 4
    grid-template-columns: repeat(4, 1fr);
    gap: 20px;

    .exam-item {
        // display: flex;
        // align-items: center;

        .exam-label {
            font-size: 14px;
            color: #333;
            // width: 50px;
            // text-align: right;
            // margin-right: 10px;
            margin-bottom: 5px;
            font-weight: bold;
        }

        .exam-content {
            flex: 1;

            :deep(.el-input-group__append),
            :deep(.el-input-group__prepend) {
                background-color: transparent;
                color: #333;
            }
        }
    }
}

.el-card {
    margin-bottom: 20px;
}
</style>