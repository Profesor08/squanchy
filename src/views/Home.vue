<template>
  <div class="home">
    <div class="text" ref="text">
      <p>
        Текст (от лат. textus — «ткань; сплетение, связь, паутина, сочетание»)
        — зафиксированная на каком-либо материальном носителе человеческая
        мысль; в общем плане связная и полная последовательность символов.
      </p>

      <p>
        Существуют две основные трактовки понятия «текст»: «имманентная»
        (расширенная, философски нагруженная) и «репрезентативная» (более
        частная). Имманентный подход подразумевает отношение к тексту как к
        автономной реальности, нацеленность на выявление его внутренней
        структуры. Репрезентативный — рассмотрение текста как особой формы
        представления знаний о внешней тексту действительности.
      </p>

      <p>
        В лингвистике термин текст используется в широком значении, включая и
        образцы устной речи. Восприятие текста изучается в рамках лингвистики
        текста и психолингвистики. Так, например, И. Р. Гальперин определяет
        текст следующим образом: «это письменное сообщение, объективированное
        в виде письменного документа, состоящее из ряда высказываний,
        объединённых разными типами лексической, грамматической и логической
        связи, имеющее определённый моральный характер, прагматическую
        установку и соответственно литературно обработанное»[1].
      </p>
    </div>
  </div>
</template>

<script>
import Az from "../lib/az/Az";

export default {
  name: "home",

  data() {
    return {
      word: "сквончить",
    };
  },

  async mounted() {
    Az.Morph.init("dicts/", (err, Morph) => {
      const capitalize = s => {
        if (typeof s !== "string") {
          return "";
        }
        return s.charAt(0).toUpperCase() + s.slice(1);
      };

      const processNode = parent => {
        Array.from(parent.childNodes).forEach(node => {
          if (node.nodeType === Node.TEXT_NODE) {
            let text = node.data;
            const tokens = Az.Tokens(text).done();
            const verbs = tokens
              .map(token => {
                const tokenText = text.substr(token.st, token.length);
                const parses = Morph(tokenText);

                return {
                  text: tokenText,
                  parses,
                  token,
                };
              })
              .filter(token => {
                for (const dict of token.parses) {
                  if (dict.tag.VERB === true) {
                    return true;
                  }

                  break;
                }
                return false;
              });

            for (const verb of verbs) {
              const word = verb.token.firstUpper
                ? capitalize(this.word)
                : this.word;
              text = text.split(verb.text).join(word);
            }

            node.nodeValue = text;
          } else if (parent) {
            processNode(node);
          }
        });
      };

      processNode(this.$refs.text);
    });
  },
};
</script>
