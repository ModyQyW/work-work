<script setup lang="ts">
import readXlsxFile from "read-excel-file";

const files = ref<File[]>([]);
const list = computedAsync(
  async () =>
    await Promise.all(
      files.value.map(async (f: File) => {
        const count = await readXlsxFile(f).then((rows) => {
          if (rows[0].some((cell) => cell.toString().toLowerCase() === "id")) {
            const index = rows[0].findIndex((cell) => {
              const value = cell.toString();
              return value.includes("工时") || value.includes("小时");
            });
            return rows
              .slice(1)
              .map((row) => row[index])
              .filter(Boolean)
              .reduce(
                (acc: number, cur) => acc + Number.parseFloat(cur.toString()),
                0
              );
          }
          return rows
            .flat(Number.POSITIVE_INFINITY)
            .filter(Boolean)
            .filter((cell) => cell.toString().includes("小时"))
            .flatMap((cell) => cell.toString().match(/\s*\d+\.\d+小时\s*/g))
            .filter((value): value is string => !!value)
            .map((string) => Number.parseFloat(string.trim().slice(0, -2)))
            .reduce((acc, cur) => acc + cur);
        });
        return {
          name: f.name,
          count,
        };
      })
    ),
  []
);
const handleFileInputChange = (event: Event) => {
  files.value = [...(event.target!.files ?? [])];
};
</script>

<template>
  <UContainer class="p-20">
    <header>
      <h1 class="text-2xl font-medium text-center">
        Read XLSX and Calculate Work Hours
      </h1>
    </header>
    <main class="mt-12 max-w-[512px] mx-auto w-4/5">
      <UInput
        type="file"
        multiple
        accept=".xlsx"
        @input="handleFileInputChange"
      ></UInput>
      <div class="mt-4"></div>
      <p>You have uploaded {{ files.length }} files.</p>
      <ul class="list-disc pl-4" v-if="list.length > 0">
        <li>Total: {{ list.reduce((acc, cur) => acc + cur.count, 0) }}h</li>
        <li v-for="item of list" :key="item.name">
          {{ item.name }} - {{ item.count }}h
        </li>
      </ul>
    </main>
  </UContainer>
</template>
