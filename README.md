# go_prof_and_trace
var cpuprofile = flag.String("cpuprofile", "", "write cpu profile to `file`")
	var memprofile = flag.String("memprofile", "", "write memory profile to `file`")

	flag.Parse()

	if *cpuprofile != "" {
		f, err := os.Create(*cpuprofile)
		if err != nil {
			log.Printf("could not create CPU profile: ", err)
		}
		defer f.Close() // error handling omitted for example
		if err := pprof.StartCPUProfile(f); err != nil {
			log.Printf("could not start CPU profile: ", err)
		}
		defer pprof.StopCPUProfile()
	}

	// ... rest of the program ...

	if *memprofile != "" {
		f, err := os.Create(*memprofile)
		if err != nil {
			log.Printf("could not create memory profile: ", err)
		}
		defer f.Close() // error handling omitted for example
		runtime.GC()    // get up-to-date statistics
		if err := pprof.WriteHeapProfile(f); err != nil {
			log.Printf("could not write memory profile: ", err)
		}
	}

	f, errBm := os.OpenFile("benchmark.log", os.O_APPEND|os.O_CREATE|os.O_WRONLY, 0644)
	if errBm != nil {
		fmt.Println(errBm)
	}
	defer func() {
		if err := f.Close(); err != nil {
			log.Printf("failed to close trace file: %v", err)
		}
	}()
	//go tool pprof -http=:8080 benchmark.log
	pprof.Lookup("heap").WriteTo(f, 0)

	//трассировка
	//go tool trace trace.log
	f, err := os.Create("trace.log")
	if err != nil {
		panic(err)
	}

	if err := trace.Start(f); err != nil {
		log.Printf("failed to start trace: %v", err)
	}
	defer trace.Stop()
