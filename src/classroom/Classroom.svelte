<script lang="ts">
    import Navbar from './Navbar.svelte';
    import Sidebar from './Sidebar.svelte';
    import ClassCard from './ClassCard.svelte';

    let { enterApp }: { enterApp: () => void } = $props();

    const classes = [
        {
            title: "AP Physics C: Mechanics",
            section: "Period 1",
            teacher: "Isaac Newton",
            bgImage: "https://gstatic.com/classroom/themes/img_read.jpg",
            avatar: "https://api.dicebear.com/7.x/avataaars/svg?seed=Isaac"
        },
        {
            title: "AP Calculus BC",
            section: "Period 2",
            teacher: "Gottfried Leibniz",
            bgImage: "https://gstatic.com/classroom/themes/img_code.jpg",
            avatar: "https://api.dicebear.com/7.x/avataaars/svg?seed=Gottfried"
        },
        {
            title: "AP English Literature",
            section: "Period 3",
            teacher: "William Shakespeare",
            bgImage: "https://gstatic.com/classroom/themes/img_bookclub.jpg",
            avatar: "https://api.dicebear.com/7.x/avataaars/svg?seed=William"
        },
        {
            title: "AP Computer Science A",
            section: "Period 4",
            teacher: "Alan Turing",
            bgImage: "https://gstatic.com/classroom/themes/img_breakfast.jpg",
            avatar: "https://api.dicebear.com/7.x/avataaars/svg?seed=Alan"
        },
        {
            title: "US History",
            section: "Period 5",
            teacher: "George Washington",
            bgImage: "https://gstatic.com/classroom/themes/img_graduation.jpg",
            avatar: "https://api.dicebear.com/7.x/avataaars/svg?seed=George"
        },
        {
            title: "Chemistry Honors",
            section: "Period 6",
            teacher: "Marie Curie",
            bgImage: "https://gstatic.com/classroom/themes/img_learnlanguage.jpg",
            avatar: "https://api.dicebear.com/7.x/avataaars/svg?seed=Marie"
        }
    ];
</script>

<div class="min-h-screen bg-white text-gray-900 font-sans">
    <Navbar onVerdisClick={enterApp} />
    <Sidebar />
    
    <main class="pt-20 px-4 md:pl-80 md:pr-8 pb-8">
        <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-6 justify-items-center sm:justify-items-start">
            {#each classes as cls}
                <ClassCard {...cls} />
            {/each}
        </div>
    </main>
</div>