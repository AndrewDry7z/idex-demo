<template>
    <div class="wrap">
        <div class="table">
            <div class="table-line">
                <div class="table-line__item">Фамилия, Имя</div>
                <div class="table-line__item">дней</div>
                <div class="table-line__item" v-for="monthName in monthsArray" v-bind:key="monthName">{{ monthName }}
                </div>
            </div>
            <template v-for="worker in staffArray">
                <div class="table-line worker-line" :key="worker.id" :id="worker.id">
                    <div class="table-line__item">
                        <div class="worker">
                            <div class="worker-photo">
                                <img :src="'./images/photo/' + worker.photo"/>
                            </div>
                            <div class="worker-name">
                                {{ worker.firstName }} <br> {{ worker.lastName }}
                            </div>
                        </div>
                    </div>
                    <div class="table-line__item">
                        {{getFinalVacationDuration(worker)}}/28
                    </div>
                    <template v-for="(monthName, index) in monthsArray">
                        <div class="table-line__item month-flex" v-bind:key="monthName">
                            <template v-for="week in getDaysArray(2019, index+1)">
                                <div class="week" v-bind:key="week">
                                    <template v-for="day in week">
                                        <div v-bind:key="day.day"
                                             :class="
                                             ['day',
                                             {'vacation-day' : isVacationDay(day, index+1, worker)},
                                             {'sickness-day' : isSicknessDay(day, index+1, worker)}]"
                                             @mouseover="haveModal(day, index+1, worker).bool ?
                                             showModal(worker, haveModal(day, index+1, worker).type,
                                             isVacationDay(day, index+1, worker)) : false"
                                             @mouseleave="haveModal(day, index+1, worker).bool ? removeModal() : false"
                                        ></div>
                                    </template>
                                </div>
                            </template>
                        </div>
                    </template>
                </div>
            </template>
        </div>
    </div>
</template>

<script>
    import "./Timeline.scss";
    import unique from "../unique";

    let getDaysArray = function (year, month) {
        let date = new Date(year, month - 1, 1);
        let result = [];

        while (date.getMonth() === month - 1) {

            result.push({
                day: date.getDate(),
                // get week number
                week: Math.round(((date - (new Date(year, 0, 0))) / (1000 * 60 * 60 * 24 * 7))) + 1,
                month: month - 1
            });

            date.setDate(date.getDate() + 1);
        }

        let weeksList = result.map((day) => {
            return day.week;
        });

        let uniqueWeeksList = unique(weeksList);
        let groupedByWeekArray = [];

        for (let weekNumber of uniqueWeeksList) {
            const grouped = groupBy(result, week => week.week);
            groupedByWeekArray.push(grouped.get(weekNumber));
        }

        return groupedByWeekArray;
    };

    function groupBy(list, keyGetter) {
        let map = new Map();
        list.forEach((item) => {
            let key = keyGetter(item);
            let collection = map.get(key);
            if (!collection) {
                map.set(key, [item]);
            } else {
                collection.push(item);
            }
        });
        return map;
    }

    function isVacationDay(day, month, worker) {
        let result;
        if (worker.vacations) {
            for (let vacation of worker.vacations) {
                if ((day.day >= vacation.startDay)
                    && (day.day < vacation.startDay + vacation.duration)
                    && (month === vacation.startMonth)) {
                    result = true;
                    sessionStorage.setItem('startDay', vacation.startDay);
                    sessionStorage.setItem('startMonth', vacation.startMonth);
                    sessionStorage.setItem('duration', vacation.duration);
                    sessionStorage.setItem('year', vacation.year);
                }
            }
        }
        return result;
    }

    function isSicknessDay(day, month, worker) {
        let result;
        if (worker.sickness) {
            for (let sickness of worker.sickness) {
                if ((day.day >= sickness.startDay)
                    && (day.day <= sickness.startDay + sickness.duration - 1)
                    && (month === sickness.startMonth)) {
                    result = true;
                    sessionStorage.setItem('startDay', sickness.startDay);
                    sessionStorage.setItem('startMonth', sickness.startMonth);
                    sessionStorage.setItem('duration', sickness.duration);
                    sessionStorage.setItem('year', sickness.year);
                }
            }
            return result;
        }
    }


    function showModal(worker, type) {
        let modal = document.createElement('div');

        //let datesArray = [];

        let startDay, startMonth, year, endDay, endMonth, absenceTypeName, duration, spanColor;

        modal.classList.add('modal');

        function showAdditionalZero(number) {
            if (number <= 9) {
                return '0';
            } else {
                return '';
            }
        }

        startDay = showAdditionalZero(sessionStorage.getItem('startDay')) + sessionStorage.getItem('startDay');
        startMonth = showAdditionalZero(sessionStorage.getItem('startMonth')) + sessionStorage.getItem('startMonth');
        duration = parseInt(sessionStorage.getItem('duration'));
        year = sessionStorage.getItem('year');
        endDay = parseInt(sessionStorage.getItem('startDay')) + duration;
        endMonth = startMonth;

        if (type === 'vacation') {
            absenceTypeName = 'отпуск';
            spanColor = '#1EBEB4';
        } else {
            absenceTypeName = 'болезнь';
            spanColor = '#F16953';
        }

        endDay = showAdditionalZero(endDay) + endDay;

        modal.innerHTML = `
        <div class="worker">
            <div class="worker-photo">
                <img src='./images/photo/${worker.photo}' alt="${worker.firstName} ${worker.lastName}">
            </div>
            <div class="worker-name">
                ${worker.firstName}<br/>${worker.lastName}
            </div>
        </div>
        <div class="vacation-info">
            <p class="vacation-info__dates">
                <span class="vacation-info__typeblock" style="background-color: ${spanColor}"></span>
                ${startDay}.${startMonth}.${year} — ${endDay}.${endMonth}.${year} (${duration} д.)
            </p>
            <p class="vacation-info__type">
                ${absenceTypeName}
            </p>
        </div>
        `;

        document.addEventListener('mousemove', (event) => {
            if (screen.width > 480) {
                if (event.clientX > (screen.width - 400)) {
                    modal.style.left = event.clientX - 300 + 'px';
                } else {
                    modal.style.left = event.clientX + 15 + 'px';
                }
                modal.style.top = event.clientY + 10 + 'px';
            }
        });

        document.querySelector('body').appendChild(modal);
    }

    function removeModal() {
        let modal = document.querySelector('.modal');
        if (modal) {
            modal.remove();
        }
    }

    function haveModal(day, month, worker) {
        let result = {};
        if (isVacationDay(day, month, worker)) {
            result.bool = true;
            result.type = 'vacation';
        } else if (isSicknessDay(day, month, worker)) {
            result.bool = true;
            result.type = 'sickness';
        }
        return result;
    }

    function getFinalVacationDuration(worker) {
        if (worker.vacations) {
            let finalDuration = 0;
            for (let vacation of worker.vacations) {
                finalDuration += parseInt(vacation.duration);
            }
            return finalDuration;
        }
    }


        export default {
        name: "Dashboard",
        data: function () {
            return {
                monthsArray: ['Янв', 'Фев', 'Мар', 'Апр', 'Май', 'Июн', 'Июл', 'Авг', 'Сен', 'Окт', "Ноя", "Дек"],
                staffArray: [
                    {
                        id: 1,
                        firstName: 'Иван',
                        lastName: 'Иванов',
                        photo: 'ivan.png',
                        vacations: [
                            {
                                id: 1,
                                startDay: 1,
                                startMonth: 2,
                                duration: 7,
                                year: 2019
                            },
                            {
                                id: 2,
                                startDay: 10,
                                startMonth: 6,
                                duration: 7,
                                year: 2019
                            },
                        ],
                        sickness: [
                            {
                                id: 1,
                                startDay: 6,
                                startMonth: 4,
                                duration: 4,
                                year: 2019
                            }
                        ]

                    },
                    {
                        id: 2,
                        firstName: 'Катерина',
                        lastName: 'Иванова',
                        photo: 'katerina.png',
                        vacations: [
                            {
                                id: 1,
                                startDay: 9,
                                startMonth: 8,
                                duration: 14,
                                year: 2019
                            }
                        ]
                    },
                    {
                        id: 3,
                        firstName: 'Иван',
                        lastName: 'Иванов',
                        photo: 'ivan.png',
                        vacations: [
                            {
                                id: 1,
                                startDay: 1,
                                startMonth: 4,
                                duration: 10,
                                year: 2019
                            },
                            {
                                id: 2,
                                startDay: 10,
                                startMonth: 11,
                                duration: 10,
                                year: 2019
                            },
                        ],
                        sickness: [
                            {
                                id: 1,
                                startDay: 22,
                                startMonth: 10,
                                duration: 7,
                                year: 2019
                            }
                        ]
                    },
                    {
                        id: 4,
                        firstName: 'Иван',
                        lastName: 'Иванов',
                        photo: 'ivan.png',
                        vacations: [
                            {
                                id: 1,
                                startDay: 16,
                                startMonth: 1,
                                duration: 14,
                                year: 2019
                            },
                            {
                                id: 2,
                                startDay: 1,
                                startMonth: 9,
                                duration: 7,
                                year: 2019
                            }
                        ]
                    }
                ],
            };
        },
        methods: {
            getDaysArray,
            isVacationDay,
            isSicknessDay,
            showModal,
            removeModal,
            haveModal,
            getFinalVacationDuration,
        },
    }
</script>